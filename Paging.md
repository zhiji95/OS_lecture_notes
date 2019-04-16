# Paging

## Memory

1. wishilist

   1. Sharing: multiple processes coexist in main memory

   2. Transparency:

      1. Processes are not aware that memory is shared
      2. Run regard less of number/locations of other processes

   3. Protection: Can not access data of OS or other processes

   4. Efficiency: should have reasonable performance

      1.  Purpose of sharing is to increase efficiency

      2. Do not waste CPU or memory resources **(fragmentation)**

         

2. Memory manage unit

   - Map program-generated address (virtual address) to hardware address (physical address) dynamically at every reference 

   - Check range and permissions 
   - Programmed by OS 

3. x86 address translation

   ![image-20190402125403170](/Users/zhiji/Library/Application Support/typora-user-images/image-20190402125403170.png)

   1. CPU generates virtual address (seg, offset) => segmentation unit:linear address => paging unit => physical memory.

   2. Segmentation: Divide virtual address space into separate logical segments; each is part of physical mem. 

   3. Segmentation hardware

      ![image-20190410114253094](/Users/zhiji/Library/Application Support/typora-user-images/image-20190410114253094.png)

4. Paging overview

   1. Goal: 
      1. **Eliminate** fragmentation due to large segments
      2. don't allocate memory that will not be used
      3. enable fine-grained sharing
   2. Paging: divide memory into fixed-sized pages for both virtual and physical memory.
   3. terminology:
      1. virtual page: page
      2. physical page: frame

5. Paging translation

   1. Address = page number + page offset

   2. Translate virtual page number to physical page number using paging table

   3. calculation:

      1. Pa = page_table[va/pg_sz/ + va%pg_sz

      2. example:

         8-bit virtual address, 10-bit physical address, each page is 64 bytes How many virtual pages? 

         – 2^8/64=4virtualpages 

         How many physical pages? – 2^10/64=16physicalpages 

         How many entries in page table? – Pagetablecontains4entries 

         Given page table = [2, 5, 1, 8], what’s the physical address for 

         virtual address 241?
          – 241=11110001b
          – 241/64=3=11b
          – 241%64=49=110001b
          – page_table[3]=8=1000b
          – Physical address = 8 * 64 + 49 = 561 = 1000110001b 

      3. m-bit virtual address, n-bit physical address, k- bit page size 

         • # of virtual pages: 2(m-k)
          • # of physical pages: 2(n-k)
          • # of entries in page table: 2(m-k) • vpn=va/2k
          • offset=va%2k
          • ppn = page_table[vpn]
          • pa=ppn*2k +offset 

6. Page protection:

   1. associating protecting bits with each virtual page in page table
   2. protection bits: 
      1. present bit: map to a valid physical page ?
      2. read/write/execute bits : can read/write/execute?
      3. user bit: can access used in the user mode?
   3. X86: PTE_P, PTE_W, PTE_U
   4. checked by MMU on each memory access

7. Page table:

   1. stored in memory

      1. Page table base register (PTBR) points to the base of page table 
      2.  OS stores base in process control block (PCB)
      3. OS switches PTBR on each context switch

   2. problem: each data access requires two memory access (extra memory access for page table)

   3. size issue: Hierarchical page table:

      break up virtual address space into multiple page tables at different levels.



## Paging in x86 and TLB

### 1. X86 page translation with 4kb pages

### 2. Avoiding extra memory access

1. Locality:

   1. Temporal: access locations accessed just now
   2. Spatial: access locations adjacent to locations accessed just now
   3. Process often needs only a small number of VPN -> PPN mappings at any moment

2. Fast-lookup hardware cache called associate memory or translation look-aside buffer (TLBs)

   1. Fast parallel search (CPU speed)
   2. Small

3. effective access time with TLB

   1. Assume memory cycle time is 1 unit time

   2. TLB lookup time: e

   3. TLB hit ratio: percentage of times that a vpn-> ppn mapping is found in TLB: a

   4. Effective Access Time (EAT):

      EAT = 2 + e + a

4. TLB and ocntext switch:

   1. option 1: flush entire TLB (cr3: special register always pointing to base address of page directory)
   2. Option 2: attach process ID to TLB entries:
      1. ASID: address space identifier
      2. MIPS, SPARC 
         1. CPU size is small
         2. page lookup is done in software
         3. put an additional column to TLB (which process own this entry)
   3. X86 "INVLPG addr" invalidates one TLB entry

5. Motivation for paging sharing:

   1. when happens:
      1. memmap
      2. Shared library

   

   1. efficient communication: processes communicate by write to shared pages
   2. memory efficiency: one copy of read-only code/data shared among process

6. Copy-on-write

   1. In `fork()`, parent and child often share significant amount of memory
   2. COW ideal exploit VA to PA indirection
      1. Instead of copying all pages, share them
      2. if either process writes to shared pages, only then is the page copied
   3. Real: used in virtually all modern OS.



## Virtual Memory

### 1. Motivation

1. Previous approach to memory management
2. Observation: locality of references
3. Implication: process only needs a small part of address soacwe at any moment

### 2. Idea

1. OS and hardware produce the illusion of disk as fast as main memory, or main memory as large as disk.
2. prococess runs when not all pages are loaded in memory
   1. only keep referenced pages in main memory
   2. keep unreferenced pages on slower, cheaper backing store
   3. 

![image-20190411123052808](/Users/zhiji/Library/Application Support/typora-user-images/image-20190411123052808.png)

### 3. page fault

1. Delect page reference:
   1. overload the present bit of page table entries
   2. if a page is on disk, clear present bit
   3. Page fault: if bit is cleared then referencing resulting in a trap into OS
   4. In OS page fault handler, check page table entry to detect if page table is caused by reference to true invalid page or page on disk.

![image-20190411123651606](/Users/zhiji/Library/Application Support/typora-user-images/image-20190411123651606.png)

4. an I/O, when OS issues an I/O, what will happen? Then the program is put to sleep, then call schedule. 



### 4. Page selection

1. Demand paging: load page on page fault.
   1. start up process with no pages loaded.
   2. Wait until a page absolutely must be in memory.
2. Request paging, user specify which pages are needed
3. Prepaging: load page before it is referenced
   1. when one page is referenced, bring in next one.
4. Thrashing

### 5. Page replacement algorithms

1. Four algorithms
   1. Optimal: throw out page that won't be used for longest time in future
      1. you can't do it (you don't know)
   2. Random: throw out a random page
   3. FIFO: throw out page that was loaded in first
      1. Problem: 
         1. ignores the access pattern
         2. fewer physical pages,fewer faults
   4. LRU: throw out page that hasn't been used in longest time (what Real OS do)
      1. advantage: with locality, 
      2. Problem: Implementation
         1. Hardware:
            1. A counter for each page
            2. Every time page is reference, save system clock into the counter of the page
            3. Page replacement: scan through pages to find the one with the oldest clock
            4. Problem: have to search all pages/ counters
         2. Software:
            1. A doubly linked list of pages
            2. everytime the page is referenced, move it to the front of list
      3. Convept vs reality:
         1. LRU is conwider to be a reasonably good algorithm
         2. Problem is in implementing it efficienty
         3. In practice, settle fort efficient approximate LRU
2. evaluate page algorithm
   1. Goal: fewest number of page faults
   2. A method: 
      1. Run algorithm
      2. computing the number of page faults.
3. Clock algorithm:
   1. Goal: remove a page that has not been referenced recently
   2. Idea: 
      1. a reference bit per page
      2. Memory bit
   3. Extension:
      1. Dirty bit: give reference to dirty pages
      2. on page reference:
         1. read: hardware sets references bit
         2. write: hardware sets dirty set
      3. page replacement:
         1. Reference = 0
         2. referenf



### 6. Current trend

1. virtual memory is less critical now:
2. Virtual to physical translation is still useful
3. Larger page sizes (even multiple page sizes)
4. Larger virtual address space
5. File I/O using the virtual memory system
