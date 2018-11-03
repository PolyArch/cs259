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
    indexing_policy = SetAssociative
```

## Step 2: Implement your SimObject in C++

Next, we need to create ```nmru.hh``` and ```nmru.cc``` which will implement our NMRU replacement policy. Importantly, every SimObject must inherit from the C++ SimObject class. Most of the time, your SimObject’s parent will be a subclass of SimObject, not SimObject itself.

For nmru.hh, we can mostly copy the code from other similar replacement policies, say LRU. In the Python file for NMRU, we used BaseSetAssoc as our parent, which we will mirror in C++. Below is the contents of ```nmru.hh```.

```
#ifndef __MEM_CACHE_TAGS_NMRU_HH__
#define __MEM_CACHE_TAGS_NMRU_HH__

#include "base/bitfield.hh"
#include "base/intmath.hh"
#include "base/logging.hh"
#include "base/statistics.hh"
#include "base/types.hh"
#include "mem/cache/cache_blk.hh"

#include "mem/cache/tags/base_set_assoc.hh"
#include "params/NMRU.hh"

class NMRU : public BaseSetAssoc
{
  public:

  /** Typedef the block type used in this class. */
  typedef CacheBlk BlkType;

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
                          std::vector<BlkType*>& evict_blks) const;
  void insertBlock(const Addr addr, const bool is_secure,
                     const int src_master_ID, const uint32_t task_ID,
                     BlkType *blk);
  void invalidate(BlkType *blk);
};

#endif // __MEM_CACHE_TAGS_NMRU_HH__
```

Again, for the implementation we can use similar code to LRU and Random
replacement policies. The basic implementation is that we track the most
recently used block by moving the last accessed block to the head of the MRU queue. On a replacement, we select a random block that is not the most recently used block. Below is the implementation in nrmu.cc:

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

NMRU::BlkType*
NMRU::accessBlock(Addr addr, bool is_secure, Cycles &lat) // , int master_id)
{
    // Accesses are based on parent class, no need to do anything special
    BlkType *blk = BaseSetAssoc::accessBlock(addr, is_secure, lat);

    if (blk != NULL) {
        // move this block to head of the MRU list
		indexingPolicy->moveToHead(blk);
    }

    return blk;
}

NMRU::BlkType*
NMRU::findVictim(Addr addr, const bool is_secure,
                         std::vector<BlkType*>& evict_blks) const
{
    BlkType *blk = BaseSetAssoc::findVictim(addr, is_secure, evict_blks);

    // if all blocks are valid, pick a replacement that is not MRU at random
    if (blk->isValid()) {
        // find a random index within the bounds of the set
        int idx = random_mt.random<int>(1, allocAssoc - 1);
        assert(idx < allocAssoc);
        assert(idx >= 0);
		const std::vector<ReplaceableEntry*> entries =
        indexingPolicy->getPossibleEntries(addr);
        blk = static_cast<BlkType*>(entries[idx]);
		evict_blks.push_back(blk);
    }

    return blk;
}

void 
NMRU::insertBlock(const Addr addr, const bool is_secure,
                     const int src_master_ID, const uint32_t task_ID,
                     BlkType *blk)
{
    BaseSetAssoc::insertBlock(addr, is_secure, src_master_ID, task_ID, blk);
	indexingPolicy->moveToHead(blk);
}

void
NMRU::invalidate(BlkType *blk)
{
    BaseSetAssoc::invalidate(blk);
    indexingPolicy->moveToTail(blk);
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

## Step 4: Add moveToHead and moveToTail utility functions

Also, we need to add moveToHead and moveToTail utility functions in set_associative.hh.

In ```src/mem/cache/tags/indexing_policies/base.hh```

```
#include "mem/cache/cache_blk.hh"
/**
 * Move the given block to the head of the given set
 */
void moveToHead(CacheBlk *blk) { }
/**
 * Move the given block to the tail of the given set
 */
void moveToTail(CacheBlk *blk) { }
```

In ```src/mem/cache/tags/indexing_policies/set_associative.hh```

```
void moveToHead(CacheBlk *blk);
void moveToTail(CacheBlk *blk);

/**
 * Swap the blocks in a set
 */
void swap(CacheBlk *blk1, CacheBlk *blk2){
  CacheBlk* temp = blk1;
  blk1=blk2;
  blk2=temp;
}

```

In ```src/mem/cache/tags/indexing_policies/set_associative.cc```

```
void SetAssociative::moveToHead(CacheBlk *blk)
{
  uint32_t set_id = extractSet(blk->tag);
  std::vector<ReplaceableEntry*> cur_set = sets[set_id];
  CacheBlk* head = static_cast<CacheBlk*>(cur_set[0]);

  // just swap with the top element
  for(const auto& entry : cur_set){
	CacheBlk* temp = static_cast<CacheBlk*>(entry);
	if(temp==blk){
	  swap(temp,head);
      DPRINTF(CacheRepl, "set %x: moving blk %x to MRU\n",
              set_id, blk->tag);
	  break;
	}
  }
}

void SetAssociative::moveToTail(CacheBlk *blk)
{
  uint32_t set_id = extractSet(blk->tag);
  std::vector<ReplaceableEntry*> cur_set = sets[set_id];
  CacheBlk* tail = static_cast<CacheBlk*>(cur_set[assoc-1]);

  // just swap with the tail element
  for(const auto& entry : cur_set){
	CacheBlk* temp = static_cast<CacheBlk*>(entry);
	if(temp==blk){
	  swap(temp,tail);
      DPRINTF(CacheRepl, "set %x: moving blk %x to MRU\n",
              set_id, blk->tag);
	  break;
	}
  }
}

```

## Step 5: Other things to make it work

Since your NMRU class extends from the BaseSetAssoc class, you need to make it's ```private``` members you want to access as ```protected```. Edit the visibility of ```extractSet``` function in ```src/mem/cache/tags/indexing_policies/set_associative.hh```.


## Step 6: Modify the config scripts to use your new SimObject

Finally, you need to create your SimObject in the config scripts. If you’re using the se.py configuration script, you can simply change the L1D cache at the end of ```se.py``` file as below:

```
for i in xrange(np):
        system.cpu[i].dcache.tags = NMRU()
```

[//]: <> (The changeset to add all of the NMRU code can be found here: [nmru-patch]({{site.baseurl}}/hws/nmru-patch). You can apply this patch by using git am.)
