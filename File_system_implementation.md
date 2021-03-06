# File System Implementation

### 1. The way to think

1. Two different aspects of them
   1. Data structure: 
   2. Access methods

### 2. Overall organization

1. Divide the disk into block. (We use just one block size)
2. User data
   1. data region: the region of disk we use for user data
3. Inode: store the information to track information about each file.
   1. inode table: holds an array of on-diesk Inodes
   2. Inodes are typically not that big
4. Allocation structures
   1. Free list: points to the first free block, then points to the next free block
   2. Bitmap:
      1. data bitmap: bitmap for data region
      2. Inode bitmap: bitmap for inode table
      3. each bit is used to indicate whether the corresponding object is free(0), in use(1)
      4. a little bit of overkill for this example
5. Superblock:
   1.  contains information about this particular file system
   2. When mounting a file system, the operating system will read the superblock first to initialize various parameters and then attach the volume to the file-system tree.

### 3. File Organization: The Inode

1. Number: each inode is implicitly referred to by a number. You can directly calculate the location of the inode on disk. 

```F#
blk = (inumber * sizeof(inode_t)) / blockSize;
sector = ((blk * blockSize) + inodeStartAddr) / sectorSize;
```

2. Inside each inode is virtually all information you need about a file (Metadata) (type, size, number of blocks allocated to it, protection information)
3. Direct pointers (inside inode): each pointer refers to one disk block that belongs to the file.
4. Multi-level index (support bigger files)
   1. Indirect pointers: points to a block that contains more pointers, each of which points to a block
   2. Inode make have fixed number of direct pointers and a single indirect pointer.
   3. Double indirect pointer: refers to a block that contains pointers to direct blocks, each of which contain pointers to data blocks. 
5. Extent: ext4 use extents instead of simple pointers
6. "Most files are small"

### 4. Directory organization

1. Directory
   1. A directory basically just contains a list of pairs (entry name, inode number). 
   2. For each file or directory in a given directory, there is a string and a number in the data block. 
   3. For each string, there may also be a length
   4. each directory have two extra entries:
      1. Dot ".": the current directory
      2. Dot-dot, "..", is the parent directory.
2. Deleting a file: `unlink` can leave an empty space in the middle of the directory, 
3. Where directories are stored: 
   1. file systems treat directories as a special type of file.
   2. A directory has an inode (type filed of the inode marked as "directory" instead of "regular file")
   3. Directory has data blocks pointed to by the inode.

### 5. Free Space Management

1. A file system must track which data blocks are free to finds space for new file or directory to allocate.
2. Two simple bitmap for free space management
3. Steps when creating a file:
   1. the file system will thus search through the bitmap for an inode that is free.
   2. allocate it to the file.
   3. file system mark the inode as used.(1)
   4. update the on-disk bitmap.

### 6. Access path: reading and writing

1. Reading a file from disk:

   1. `open("/foo/bar", O RDONLY)` (the amount of I/O generated by open proportional to the length of pathname)

      the file system first needs to find the inode for the file `bar` , to obtain some basic information about the file

      1. traverse the pathname and locate the desired inode
      2. in most UNIX file system, the root inumber is 2.
      3. read the inode of the root directory (read the block that contains inode number 2). All traversals begin at the root of the file system, in the root directory.
      4. look inside the inode to find the pointers to data blocks. Which contains he contents of root directory. 
      5. File system will thus use these on-disk pointers to read through the directory. (Look for an entry for `foo`). FS found the inode number of `foo`
      6. Recursively traverse the pathname util the desired inode is found. (Find the inode number of `bar`)
      7. read `bar`'s inode into memory. FS does a final permission check, allocates a file descriptor for this process in the per-process open-file table.

   2. `read`

      1. read the first block of the file, consulting the inode to find the location of such a block.
      2. may also update the inode with a new last-access time.
      3. further update the in-memory open file table for this file descriptor. Update the file offset such that the next read will read the second file block.

   3. `close`

      1. the file descriptor should be deallocated

2. Writing to disk.

   1. open
   2. `write` ()
      1. logically generates 5 I/Os
         1. read the data bitmap
         2. write the bitmap
         3. read the inode
         4. write the inode
         5. Write the actual block itself
      2. Allocate a block:
         1. Decide which block to allocate
         2. update other structures accordingly.

### 7. Caching and Buffering

1. LRU: allocated at boot time to be roughly 10% of total memory. 
2. Dynamic Partitioning:
   1. Unified page cache;
3. Write buffering: 
   1. Benefits:
      1. Delaying writes, the file system can batch some updates into a smaller. 
      2. the system can schedule the subsequent I/Os and thus increase performance.
      3. some writes are avoid altogether by delaying them. (Create a file and delay it)
   2. trade off: 
      1. if the system crashes before the updates have been propagated to disk, the updates are lost. Longer the write buffer, the more updates will be lost.
      2. if the write buffer is longer, the performance will be increased.
   3. Avoid data loss: `sync()` : 
      1. using direct I/O interfaces that work around the cache
      2. using the raw disk interface and avoiding the file system altogether.
