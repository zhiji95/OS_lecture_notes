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

   1. option 1: flush entire TLB
   2. Option 2: attach process ID to TLB entries:
      1. ASID: address space identifier
      2. MIPS, SPARC
   3. X86 "INVLPG addr" invalidates one TLB entry

5. Motivation for paging sharing:

   1. efficient communication: processes communicate by write to shared pages
   2. memory efficiency: one copy of read-only code/data shared among process

6. Copy-on-write

   1. In `fork()`, parent and child often share significant amount of memory
   2. COW ideal exploit VA to PA indirection
      1. Instead of copying all pages, share them
      2. if either process writes to shared pages, only then is the page copied
   3. Real: used in virtually all modern OS.

