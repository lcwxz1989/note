
# _

## The Abstraction: Address Spaces

### Mechanism: Address Translation

Efficiency and control together are two of the
main goals of any modern operating system

The generic technique we will use, which you can consider an addition
to our general approach of limited direct execution, is something that is
referred to as hardware-based address translation, or just address translation for short

### Dynamic (Hardware-based) Relocation

Specifically, we’ll need two hardware registers within each CPU: one
is called the base register, and the other the bounds (sometimes called a
limitregister)

physical address = virtual address + base

Because this relocation of the address happens at runtime, and because we can move address spaces even after the process has started running, the technique is often referred to as dynamic relocation

### MMU

memory management unit (MMU); as we develop more sophisticated memorymanagement techniques, we will be adding more circuitry to the MMU

#### Hardware Support: A Summary

#### Segmentation: Generalized Base/Bounds

only used memory is allocated space
in physical memory, and thus large address spaces with large amounts of
unused address space (which we sometimes call sparse address spaces)
can be accommodated

#### with one critical difference: it grows backwards

Instead of
just base and bounds values, the hardware also needs to know which way
the segment grows

#### Support for Sharing

 In particular, code
sharing is common and still in use in systems today
To support sharing, we need a little extra support from the hardware,
in the form of protection bits
 Basic support adds a few bits per segment,
indicating whether or not a program can read or write a segment, or perhaps execute code that lies within the segment.

#### OS Support

segmentation raises a number of new issues for the operating system

The first is an old one: what should the OS do on a context
switch? the segment registers must be saved and restored
The second is OS interaction when segments grow (or perhaps shrink)
The last, and perhaps most important, issue is managing free space in
physical memory. compact physical memory
by rearranging the existing segments, including classic algorithms like best-fit (which keeps a list of free spaces
and returns the one closest in size that satisfies the desired allocation to
the requester), worst-fit, first-fit, and more complex schemes like buddy algorithm

#### Free-Space Management

##### Low-level Mechanisms

1. Splitting and Coalescing
2. Tracking The Size Of Allocated Regions
   The header minimally contains the size of the allocated region (in this case, 20); it may also contain additional pointers to speed up deallocation, a magic number to provide additional integrity checking, and other information
3. Embedding A Free List

#### Paging: Introduction

The first approach is to chop things up into variable-sized pieces, as we saw with segmentation in virtual memory

to chop up space into fixed-sized pieces. In virtual memory, we call this idea paging
we view physical memory as an array of fixed-sized slots called page frames

page table: To record where each virtual page of the address space is placed in physical memory, the operating system usually keeps a per-process data structure known as a page table
virtual address that the process generated, we have to first split it into two components: the virtual page number (VPN), and the offset within the page
When a process generates a virtual address, the OS and hardware must combine to translate it into a meaningful physical address

0 1 0 1 0 1
VPN offset
1 1 1 0 1 0 1
Address
Translation
PFN offset
Virtual
Address
Physical
Address
Figure 18.3: The Address Translation Process

##### Where Are Page Tables Stored

Page tables can get terribly large, much bigger than the small segment
table or base/bounds pair we have discussed previously

This virtual address splits into a 20-bit VPN and 12-bit offset (recall that 10 bits would
be needed for a 1KB page size, and just add two more to get to 4KB).

Because page tables are so big, we don’t keep any special on-chip hardware in the MMU to store the page table of the currently-running process

Let’s assume for now that the page tables live in physical memory that
the OS manages

##### What’s Actually In The Page Table

31 30 29 28 27 26 25 24 23 22 21 20 19 18 17 16 15 14 13 12 11 10 9 8 7 6 5 4 3 2 1 0
PFN G
PAT
D
A
PCD
PWT
U/S
R/W
P
Figure 18.5: An x86 Page Table Entry (PTE)

protection bits: indicating whether the page could be read from, written to, or executed from
present bit: indicates whether this page is in physical memory or on disk (i.e., it has been swapped out)
dirty bit: is also common, indicating whether the page has been modified: since it was brought into memory reference bit (a.k.a. accessed bit) is sometimes used to track whether a page has been accessed, and is useful in determining which pages are popular and thus should be kept in memory; such knowledge is critical during page replacement

##### Paging: Also Too Slow

```c
page-table base register

// Extract the VPN from the virtual address
VPN = (VirtualAddress & VPN_MASK) >> SHIFT

// Form the address of the page-table entry (PTE)
PTEAddr = PTBR + (VPN * sizeof(PTE))

// Fetch the PTE
PTE = AccessMemory(PTEAddr)

// Check if process can access the page
if (PTE.Valid == False)
    RaiseException(SEGMENTATION_FAULT)
else if (CanAccess(PTE.ProtectBits) == False)
    RaiseException(PROTECTION_FAULT)
else
// Access is OK: form physical address and fetch it
    offset = VirtualAddress & OFFSET_MASK
    PhysAddr = (PTE.PFN << PFN_SHIFT) | offset
    Register = AccessMemory(PhysAddr)
```

Extra memory references are costly, and in this case will likely slow down the process by a factor of two or more

Q: why the page table is linear
    because of without linear search for, we can find item by address plus by index

### Paging: Faster Translations (TLBs)

To speed address translation, we are going to add what is called (for historical reasons [CP78]) a translation-lookaside buffer, or TLB. A TLB
is part of the chip’s memory-management unit (MMU)

#### Who Handles The TLB Miss?

 important details
 First, when
returning from a TLB miss-handling trap, the hardware must resume execution at the instruction that caused the trap ; this retry thus lets the in struction run again
Second, when running the TLB miss-handling code, the OS needs to be
extra careful not to cause an infinite chain of TLB misses to occur.

#### HOW TO MANAGE TLB CONTENTS ON A CONTEXT SWITCH

Q: When context-switching between processes, the translations in the TLB for the last process are not meaningful to the about-to-be-run process. What should the hardware or OS do in order to solve this problem?
Q: If the OS switches between processes frequently, this cost may be high
Q: As an aside, you may also have thought of another case where two entries of the TLB are remarkably similar

1. flush the TLB on context switches: e flush could be enacted when the page-table base register is changed.  
2. provide an address space identifier (ASID) field in the TLB
3. . Sharing of code pages (in binaries, or shared libraries) is useful as it reduces the number of physical pages in use

#### Replacement Policy

1. cache replacement
    evict the least-recently-used or LRU entry
    random policy

#### A Real TLB Entry

0000000000111111111 1 2222 22222233
0123456789012345678 9 0123 45678901
VPN                 G       ASID
PFN C D V
Figure 19.4: A MIPS TLB Entry

global bit (G): which is used for pages that are globally-shared among processes
8-bit ASID: which the OS can use to distinguish between address spaces
3 Coherence (C) bits: which determine how a page is cached by the hardware
a dirty bit: which is marked when the page has been written to
a valid bit: which tells the hardware if there is a valid translation present in the entry

#### hardware-managed TLB

#### software-managed TLB

### Paging: Smaller Tables

page tables are too big and thus consume too much memory

#### Simple Solution: Bigger Pages

#### Hybrid Approach: Paging and Segments

most of the page table is unused, full of invalid entries
In the hardware, assume that there are thus three base/bounds pairs, one each for code, heap, and stack. In our hybrid, we still have those structures in the MMU;

Q:  approach is not without problems
    First, it still requires us to use segmentation; as we discussed before, segmentation is not quite as flexible as we would like, as it assumes a certain usage pattern of the address space
    Second, this hybrid causes external fragmentation to arise again

#### Multi-level Page Tables

get rid of all those invalid regions in the page table instead of keeping them all in memory

Multi-level Page Table
200
PFN
201       1 rx 12
 Linear Page Table
PTBR 201 PDBR
  PFN
1 rx 12
1rx 13
0-- 0-
PFN
     1
0 -
1rx 13 0-- 1rw 100
[Page 1 of PT: Not Allocated]
[Page 2 of PT: Not Allocated]
0-- 0-- 1 rw 86 1 rw 15
        1rw 100 0-- 0-- 0-- 0-- 0-- 0-- 0-- 0-- 0-- 0-- 1 rw 86 1 rw 15
1 204
The Page Directory
                          OPERATING SYSTEMS [VERSION 1.01]
Figure 20.3: Linear (Left) And Multi-Level (Right) Page Tables

it just makes parts of the linear page table disappear (freeing those frames for other uses), and tracks which pages of the page table are allocated with the page directory
The page directory, in a simple two-level table, contains one entry per page of the page table. It consists of a number of page directory entries (PDE). A PDE (minimally) has a valid bit and a page frame number (PFN), similar to a PTE
advantages:
    First, and perhaps most obviously, the multi-level ta- ble only allocates page-table space in proportion to the amount of address space you are using;
    Second, if carefully constructed, each portion of the page table fits neatly within a page, making it easier to manage memory

trade-off
    there is a cost to multi-level tables; on a TLB miss, two loads from memory will be required to get the right translation information from the page table (one for the page directory, and one for the PTE itself), in contrast to just one load with a linear page table. Thus, the multi-level table is a small example of a time-space trade-off
    Another obvious negative is complexity

VPN offset
13 12 11 10 9 8 7 6 5 4 3 2 1 0
Page Directory Index

page-directory index: PDEAddr = PageDirBase + (PDIndex \* sizeof(PDE))
page-table index: PTEAddr = (PDE.PFN << SHIFT) + (PTIndex \* sizeof(PTE))
physical address: PhysAddr = (PTE.PFN << SHIFT) + offset

#### More Than Two Levels

Remember our goal in constructing a multi-level page table: to make each piece of the page table fit within a single page
assume we have a 30-bit virtual address space, and a small (512 byte) page. Thus our virtual address has a 21-bit virtual page number component and a 9-bit offset

VPN offset
2928272625242322212019181716151413121110 9 8 7 6 5 4 3 2 1 0 PD Index 0 PD Index 1 Page Table Index

#### Inverted Page Tables

we keep a single page table that has an entry for each physical page of the system. The entry tells us which process is using this page, and which virtual page of that process maps to this physical page

### Beyond Physical Memory: Mechanisms

In modern systems, this role is usually served by a hard disk drive. Thus, in our memory hierarchy, big and slow hard drives sit at the bottom, with memory just above

#### Swap Space

#### The Present Bit

swap space, because we swap pages out of memory to it and swap pages into memory from it. To do so, the OS will need to remember the disk address of a given page

present bit: If the present bit is set to one, it means the page is present in physical memory and everything proceeds as above; if it is set to zero, the page is not in memory but rather on disk somewhere
page fault: The act of accessing a page that is not in physical memory is commonly referred to. The appropriately-named OS page-fault handler runs to determine what to do

Q: how will the OS know where to find the desired page?
    When the OS receives a page fault for a page, it looks in the PTE to find the address, and issues the request to disk to fetch the page into memory

#### The Page Fault

##### Page Fault Control Flow

Step:

1. First, the OS must find a physical frame for the soon-to-be-faulted-in page to reside within; if there is no such page, we’ll have to wait for the replacement algorithm to run and kick some pages out of memory
2. With a physical frame in hand, the handler then issues the I/O request to read in the page from swap space
3. When the disk I/O completes, the OS will then update the page table to mark the page as present, update the PFN field of the page-table entry (PTE) to record the in-memory location of the newly-fetched page
4. retry the instruction
5. This next attempt may generate a TLB miss, which would then be serviced and update the TLB with the translation
6. Finally, a last restart would find the translation in the TLB and thus proceed to fetch the desired data or instruction from memory at the translated physical address

#### What If Memory Is Full

##### page-replacement policy

#### When Replacements Really Occur

To keep a small amount of memory free, most operating systems thus have some kind of high watermark (HW) and low watermark (LW) to help decide when to start evicting pages from memory. a background thread that is responsible for freeing memory runs. The thread evicts pages until there are HW pages available

### Beyond Physical Memory: Policies

#### Cache Management

our goal in picking a replacement policy for this cache is to minimize the number of cache misses

average memory access time (AMAT):
    AMAT = TM +(PMiss ·TD)

#### The Optimal Replacement Policy

To better understand how a particular replacement policy works, it would be nice to compare it to the best possible replacement policy

simple (but, unfortunately, difficult to implement!) approach that replaces the page that will be accessed furthest in the future is the optimal policy, resulting in the fewest-possible cache misses

#### A Simple Policy: FIFO

FIFO has one great strength: it is quite simple to implement
Comparing FIFO to optimal, FIFO does notably worse

#### Another Simple Policy: Random

Random has properties similar to FIFO; it is simple to implement
How Random does depends on the luck of the draw

#### Using History: LRU

As we did with scheduling policy, to improve our guess at the future, we once again lean on the past and use history as our guide
frequency: if a page has been accessed many times, perhaps it should not be replaced as it clearly has some value
recency of access: the more recently a page has been accessed, perhaps the more likely it will be accessed again

a family of simple historically-based algorithms are born. The Least-Frequently-Used (LFU) policy replaces the least-frequently- used page when an eviction must take place. Similarly, the Least-Recently- Used (LRU) policy replaces the least-recently-used page

#### examine more complex workloads

1. each reference is to a random page within the set of accessed pages
    First, when there is no locality in the workload, it doesn’t matter much which realistic policy you are using with the hit rate exactly determined by the size of the cache
    Second, when the cache is large enough to fit the entire workload, it also doesn’t matter which policy you use
    Finally, you can see that optimal performs noticeably better than the realistic policies

2. “80-20” workload, which exhibits locality: 80% of the references are made to 20% of the pages, the remaining 20% of the references are made to the re- maining 80% of the pages
   both random and FIFO do rea- sonably well, LRU does better, as it is more likely to hold onto the hot pages
   LRU’s improvement over Random and FIFO really that big of a deal? The answer, as usual, is “it depends.”

3. “looping sequen- tial” workload
   represents a worst- case for both LRU and FIFO. These algorithms, under a looping-sequential workload, kick out older pages
   Random fares notably better, not quite ap- proaching optimal, but at least achieving a non-zero hit rate. Turns out that random has some nice properties

#### Approximating LRU

LRU. To keep track of which pages have been least- and most-recently used, the system has to do some accounting work on every memory reference. Clearly, without great care, such accounting could greatly reduce performance

Imagine all the pages of the system arranged in a circular list. A clock hand points to some particular page to begin with (it doesn’t really matter which). When a replacement must occur, the OS checks if the currently-pointed to page P has a use bit of 1 or 0. If 1, this implies that page P was recently used and thus is not a good candidate for replacement. Thus, the use bit for P is set to 0 (cleared), and the clock hand is incremented to the next page (P + 1). The algorithm continues until it finds a use bit that is set to 0

#### Considering Dirty Pages

if a page has been modified and is thus dirty, it must be written back to disk to evict it, which is expensive. If it has not been modified (and is thus clean), the eviction is free. To support this behavior, the hardware should include a modified bit (a.k.a. dirty bit)

#### Other VM Policies

prefetching:  the OS could guess that a page is about to be used, and thus bring it in ahead of time
clustering or simply grouping of writes: collect a number of pending writes together in memory and write them to disk in one (more efficient) write
