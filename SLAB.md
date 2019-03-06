# Slab Layer

1. Introduction:

   1. Free List: 
      1. contains a block of available, already allocated data structures.
      2. Cache frequently used type of object
      3. When code requires a new instance of a data structure, it can grab one from free list.
      4. When code nologer need,  it returned to the free list instead of deallocated. 
      5. Problem: no global control
      6. Solution: provide slab layer acting as a generic data structure-caching layer.
   2. Slab allocator tenets

2. Design:

   1. Caches: 
      1. Store different type of object (One cache per type)
      2. slab layer divides different objects into groups called caches
   2. Slab
      1. are composed of one ro more physically contiguous pages (Typically one)
      2. full slab: no free objects
      3. Empty slab: no allocated objects
   3. `inode` 
   4. Struct slab

   ```c
   struct slab {
   	struct list_head list; /* full, partial, or empty list */
       unsigned long void unsigned int
       colouroff; /* offset for the slab coloring */ 
       s_mem; /* first object in the slab */ 
       inuse; /* allocated objects in the slab */
       kmem_bufctl_t free; /* first free object, if any */
   };
   ```

   5. struct cache: kmem_cache
   6. Create: __get_free_pages()
      1. Free memory: kmem_freepages()

3. SLAB Allocator interface

```c
struct kmem_cache * kmem_cache_create(
    const char *name, //the name
    size_t size, // size of each element
    size_t align, // the offset of the first object withn a slab
    unsigned long flags, // Specifies optional settings
    void (*ctor)(void *)); //

int kmem_cache_destroy(struct kmem_cache *cachep)
```

4. Allocating form the Cache

```c
void * kmem_cache_alloc(struct kmem_cache *cachep, gfp_t flags)
void kmem_cache_free(struct kmem_cache *cachep, void *objp)
```

5. Example:

```c
task_struct_cachep = kmem_cache_create(
    “task_struct”, 
    sizeof(struct task_struct),
    ARCH_MIN_TASKALIGN, 
    SLAB_PANIC | SLAB_NOTRACK, NULL);

```

