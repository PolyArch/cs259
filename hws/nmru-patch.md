```
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
```
