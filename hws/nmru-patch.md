---
layout: post
title: NMRU patch
---

``` diff
diff --git a/build_opts/X86 b/build_opts/X86
index 64d163c..36c573a 100644
--- a/build_opts/X86
+++ b/build_opts/X86
@@ -1,3 +1,3 @@
 TARGET_ISA = 'x86'
-CPU_MODELS = 'AtomicSimpleCPU,O3CPU,TimingSimpleCPU'
+CPU_MODELS = 'MinorCPU,AtomicSimpleCPU,O3CPU,TimingSimpleCPU'
 PROTOCOL = 'MI_example'
diff --git a/configs/example/se.py b/configs/example/se.py
index 804ef02..ff11fc7 100644
--- a/configs/example/se.py
+++ b/configs/example/se.py
@@ -264,6 +264,8 @@ else:
     system.system_port = system.membus.slave
     CacheConfig.config_cache(options, system)
     MemConfig.config_mem(options, system)
+    for i in xrange(np):
+        system.cpu[i].dcache.tags = NMRU()

 root = Root(full_system = False, system = system)
 Simulation.run(options, root, system, FutureClass)
diff --git a/src/mem/cache/tags/SConscript b/src/mem/cache/tags/SConscript
index 1a3780f..c88b375 100644
--- a/src/mem/cache/tags/SConscript
+++ b/src/mem/cache/tags/SConscript
@@ -37,3 +37,4 @@ Source('base_set_assoc.cc')
 Source('fa_lru.cc')
 Source('sector_tags.cc')
 Source('skewed_assoc.cc')
+Source('nmru.cc')
diff --git a/src/mem/cache/tags/Tags.py b/src/mem/cache/tags/Tags.py
index 781ac8e..b492ef3 100644
--- a/src/mem/cache/tags/Tags.py
+++ b/src/mem/cache/tags/Tags.py
@@ -99,3 +99,8 @@ class FALRU(BaseTags):
 class SkewedAssoc(BaseSetAssoc):
     type = 'SkewedAssoc'
     cxx_header = "mem/cache/tags/skewed_assoc.hh"
+
+class NMRU(BaseSetAssoc):
+    type = 'NMRU'
+    cxx_header = "mem/cache/tags/nmru.hh"
+
diff --git a/src/mem/cache/tags/base_set_assoc.hh b/src/mem/cache/tags/base_set_assoc.hh
index 8e4657f..9a0e12c 100644
--- a/src/mem/cache/tags/base_set_assoc.hh
+++ b/src/mem/cache/tags/base_set_assoc.hh
@@ -315,7 +315,8 @@ class BaseSetAssoc : public BaseTags
         return false;
     }

-  private:
+  // private:
+  protected:
     /**
      * Calculate the set index from the address.
      *
diff --git a/src/mem/cache/tags/nmru.cc b/src/mem/cache/tags/nmru.cc
new file mode 100644
index 0000000..7f7bf22
--- /dev/null
+++ b/src/mem/cache/tags/nmru.cc
@@ -0,0 +1,78 @@
+/**
+ * @file
+ * Definitions of a NMRU tag store.
+ */
+
+#include "mem/cache/tags/nmru.hh"
+
+#include "base/random.hh"
+#include "debug/CacheRepl.hh"
+#include "mem/cache/base.hh"
+
+NMRU::NMRU(const Params *p)
+    : BaseSetAssoc(p)
+{
+}
+
+BaseSetAssoc::BlkType*
+NMRU::accessBlock(Addr addr, bool is_secure, Cycles &lat) // , int master_id)
+{
+    // Accesses are based on parent class, no need to do anything special
+    BlkType *blk = BaseSetAssoc::accessBlock(addr, is_secure, lat); // , master_id);
+
+    if (blk != NULL) {
+        // move this block to head of the MRU list
+        sets[blk->set].moveToHead(blk);
+        DPRINTF(CacheRepl, "set %x: moving blk %x (%s) to MRU\n",
+                blk->set, regenerateBlkAddr(blk),
+                is_secure ? "s" : "ns");
+    }
+
+    return blk;
+}
+
+BaseSetAssoc::BlkType*
+NMRU::findVictim(Addr addr, const bool is_secure,
+                         std::vector<CacheBlk*>& evict_blks) const
+{
+    BlkType *blk = BaseSetAssoc::findVictim(addr, is_secure, evict_blks);
+
+    // if all blocks are valid, pick a replacement that is not MRU at random
+    if (blk->isValid()) {
+        // find a random index within the bounds of the set
+        int idx = random_mt.random<int>(1, assoc - 1);
+        assert(idx < assoc);
+        assert(idx >= 0);
+        blk = sets[extractSet(addr)].blks[idx];
+
+        DPRINTF(CacheRepl, "set %x: selecting blk %x for replacement\n",
+                blk->set, regenerateBlkAddr(blk));
+    }
+
+    return blk;
+}
+
+void
+NMRU::insertBlock(PacketPtr pkt, BlkType *blk)
+{
+    BaseSetAssoc::insertBlock(pkt, blk);
+
+    int set = extractSet(pkt->getAddr());
+    sets[set].moveToHead(blk);
+}
+
+void
+NMRU::invalidate(BlkType *blk)
+{
+    BaseSetAssoc::invalidate(blk);
+
+    // should be evicted before valid blocks
+    int set = blk->set;
+    sets[set].moveToTail(blk);
+}
+
+NMRU*
+NMRUParams::create()
+{
+    return new NMRU(this);
+}
diff --git a/src/mem/cache/tags/nmru.hh b/src/mem/cache/tags/nmru.hh
new file mode 100644
index 0000000..d39ffae
--- /dev/null
+++ b/src/mem/cache/tags/nmru.hh
@@ -0,0 +1,35 @@
+#ifndef __MEM_CACHE_TAGS_NMRU_HH__
+#define __MEM_CACHE_TAGS_NMRU_HH__
+
+#include "mem/cache/tags/base_set_assoc.hh"
+#include "params/NMRU.hh"
+
+class NMRU : public BaseSetAssoc
+{
+public:
+/** Convenience typedef. */
+typedef NMRUParams Params;
+
+  /**
+   *      * Construct and initialize this tag store.
+   *           */
+  NMRU(const Params *p);
+
+  /**
+   *      * Destructor
+   *           */
+  ~NMRU() {}
+
+  /**
+   *      * Required functions for this subclass to
+   *      implement
+   *           */
+  BlkType* accessBlock(Addr addr, bool is_secure, Cycles &lat);
+  // ,int context_src);
+  BlkType* findVictim(Addr addr, const bool is_secure,
+                         std::vector<CacheBlk*>& evict_blks) const;
+  void insertBlock(PacketPtr pkt, BlkType *blk);
+  void invalidate(BlkType *blk);
+};
+
+#endif // __MEM_CACHE_TAGS_NMRU_HH__
```
