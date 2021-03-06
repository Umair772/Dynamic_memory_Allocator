DESCRIPTION:
------------

We have implemented a simple memory allocator that uses explicit free list to keep track of free blocks in allocated heap memory. It uses a doubly linked list to keep tracks of all the free blocks, first-fit search algorithm to search for the required free block and immediate boundary tag coalescing to coalesce adjoining free blocks. This is much faster than implicit list method because we don't traverse allocated blocks and only work on free memory blocks.

DESIGN:
-------

We have created a linked list "freeListPtr" that points to the starting of this linked list of all free blocks. When we allocate a block, we delete it from this linked list by using function free_list_delete() and when we free an allocated block, we insert it to the free list using free_list_add() function.

We use the 4 byte word each to store the addresses of previous and next free block in the payload area of free block. These pointers make sure we only traverse the free blocks and not the allocated blocks. The efficiency of this search algorithm increases as the number of free blocks decrease. This will increase the throughput of allocation to a great extent. If no fit is found, we extend the heap size.

A Free Block:
------------
			
	 -------------------------------------------------------------
	|        |          |          |                    |         |
        | HEADER | PREV_PTR | NEXT_PTR |                    |  FOOTER |                            
        |        |          |          |                    |         |
         -------------------------------------------------------------

Functions used:
---------------

1. mm_init() initializes the heaplist and initializes the freeListPtr.
	
2. mm_malloc() allocates the requested size that is required by the payload. Adding header, footer and alignment according to DSIZE, we modify size to asize to include this. Find fit is called to find a suitable block and if it returns successfully, we allocate the block and remove it from free list. Else, we extend the heap size by calling extend_heap.

3. mm_free() frees the allocated block by  setting the allocated bit to zero, editing the HEADER and FOOTER and coalescing the free blocks.

4. mm_realloc() reallocates the current block according the new size requirements.
   If the requested size is greater than oldsize, it tries to extend the current memory block and if it fails, it allocates a new block from the free_list of size newsize, copies old data from the old block and the frees the old block and adds it to free_list. If newsize is less than oldsize, it shrinks the old block to newsize by changing the HEADER and FOOTER and frees the remaining space.

5.coalesce() Performs boundary tag coalescing checking all possible cases. Returns the address of the coalesced block.

6.extend_heap() This is used to extend the existing heap if find_fit() fails. It extends the heapsize by (extendsize/ WSIZE) words.

7. find_fit() finds a fit for a block with "asize" bytes.  Returns that block's address or NULL if no suitable block was found. 

8. place() calculates the current size of the block. If the difference between current size of the block and requested size is more than 16 bytes then it splits the block into two. The address of the original block is removed from the free list. The first part of the old block is made allocated while the second part is made free and its address is stored in the free list. If difference is less than 16 then it just removes the address of the block from the free list and makes the block allocated.

9.free_list_add()  this adds the free block to the free_list according to the LIFO policy.

10. free_list_delete() this removes the free block from the free_list


RESULT
------
Perf Index = 31/40 (util) + 60/60 (thru) = 91/100
