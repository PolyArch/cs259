---
layout: post
title: NMRU tutorial
---

## Overview

In this tutorial we will walk though how to create a simple SimObject. As an example, we are going to create a new cache replacement policy, specifically, NMRU, not most recently used. After this chapter you should be able to create new SimObjects, instantiate them in configuration files, and run simulations with your new objects.

## Step 1: Create a Python class for your new SimObject

Each SimObject has a Python class which is associated with it. This Python class describes the parameters of your SimObject that can be controlled from the Python configuration files. For our NMRU cache tags, we are just going to inherit all of the parameters from the BaseSetAssoc tags. Thus, we simply need to declare a new class for our SimObject and set it’s name and the C++ header that will define the C++ class for the SimObject.

We edit the file, ```Tags.py```, in ```src/mem/cache/tags/```

```
class NMRU(BaseSetAssoc):
    type = 'NMRU'
    cxx_header = "mem/cache/tags/nmru.hh"
```

## Step 2: Implement your SimObject in C++

Next, we need to create ```nmru.hh``` and ```nmru.cc``` which will implement our NMRU replacement policy. Importantly, every SimObject must inherit from the C++ SimObject class. Most of the time, your SimObject’s parent will be a subclass of SimObject, not SimObject itself.

For nmru.hh, we can mostly copy the code from other similar replacement policies, say LRU. In the Python file for NMRU, we used BaseSetAssoc as our parent, which we will mirror in C++. Below is the contents of ```nmru.hh```.

```
#ifndef __MEM_CACHE_TAGS_NMRU_HH__
#define __MEM_CACHE_TAGS_NMRU_HH__

#include "mem/cache/tags/base_set_assoc.hh"
#include "params/NMRU.hh"

class NMRU : public BaseSetAssoc
{
public:
/** Convenience typedef. */
typedef NMRUParams Params;

  /**
   *      * Construct and initialize this tag store.
   *           */
  NMRU(const Params *p);

  /**
   *      * Destructor
   *           */
  ~NMRU() {}

  /**
   *      * Required functions for this subclass to
   *      implement
   *           */
  BlkType* accessBlock(Addr addr, bool is_secure, Cycles &lat);
  BlkType* findVictim(Addr addr, const bool is_secure,
                         std::vector<CacheBlk*>& evict_blks) const;
  void insertBlock(PacketPtr pkt, BlkType *blk);
  void invalidate(BlkType *blk);
};

#endif // __MEM_CACHE_TAGS_NMRU_HH__

```

Again, for the implementation we can use similar code to LRU and Random replacement policies. The basic implementation is that we track the most recently used block by moving the last accessed block to the head of the MRU queue. On a replacement, we select a random block that is not the most recently used block. Below is the implementation in ```nrmu.cc```:

```
/**
 * @file
 * Definitions of a NMRU tag store.
 */

#include "mem/cache/tags/nmru.hh"

#include "base/random.hh"
#include "debug/CacheRepl.hh"
#include "mem/cache/base.hh"

NMRU::NMRU(const Params *p)
    : BaseSetAssoc(p)
{
}

BaseSetAssoc::BlkType*
NMRU::accessBlock(Addr addr, bool is_secure, Cycles &lat) // , int master_id)
{
    // Accesses are based on parent class, no need to do anything special
    BlkType *blk = BaseSetAssoc::accessBlock(addr, is_secure, lat); // , master_id);

    if (blk != NULL) {
        // move this block to head of the MRU list
        sets[blk->set].moveToHead(blk);
        DPRINTF(CacheRepl, "set %x: moving blk %x (%s) to MRU\n",
                blk->set, regenerateBlkAddr(blk),
                is_secure ? "s" : "ns");
    }

    return blk;
}

BaseSetAssoc::BlkType*
NMRU::findVictim(Addr addr, const bool is_secure,
                         std::vector<CacheBlk*>& evict_blks) const
{
    BlkType *blk = BaseSetAssoc::findVictim(addr, is_secure, evict_blks);

    // if all blocks are valid, pick a replacement that is not MRU at random
    if (blk->isValid()) {
        // find a random index within the bounds of the set
        int idx = random_mt.random<int>(1, assoc - 1);
        assert(idx < assoc);
        assert(idx >= 0);
        blk = sets[extractSet(addr)].blks[idx];
        DPRINTF(CacheRepl, "set %x: selecting blk %x for replacement\n",
                blk->set, regenerateBlkAddr(blk));
    }

    return blk;
}

void
NMRU::insertBlock(PacketPtr pkt, BlkType *blk)
{
    BaseSetAssoc::insertBlock(pkt, blk);

    int set = extractSet(pkt->getAddr());
    sets[set].moveToHead(blk);
}

void
NMRU::invalidate(BlkType *blk)
{
    BaseSetAssoc::invalidate(blk);

    // should be evicted before valid blocks
    int set = blk->set;
    sets[set].moveToTail(blk);
}

NMRU*
NMRUParams::create()
{
    return new NMRU(this);
}
```

## Step 3: Register the C++ file

Each SimObject must be registered with SCons so that its Params object and Python wrapper is created. Additionally, we also have to tell SCons which C++ files to compile. To do this, modify the SConscipt file in the directory that your SimObject is in. For each SimObject, add a call to SimObject and for each source file add a call to Source. In this example, you need to add the following to ```src/mem/cache/tags/SConscript```:

```
Source('nmru.cc')
```

## Step 4: Other things to make it work

Since your NMRU class extends from the BaseSetAssoc class, you need to make it's ```private``` members you want to access as ```protected```. Edit the visibility of ```extractSet``` function in ```src/mem/cache/tags/SConscript```.

## Step 5: Modify the config scripts to use your new SimObject

Finally, you need to create your SimObject in the config scripts. If you’re using the se.py configuration script, you can simply change the L1D cache at the end of ```se.py``` file as below:

```
for i in xrange(np):
        system.cpu[i].dcache.tags = NMRU()
```

The changeset to add all of the NMRU code can be found here: [nmru-patch]({{site.baseurl}}/hws/nmru-patch). You can apply this patch by using git am.
