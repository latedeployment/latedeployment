---
date: 2015-02-19T19:50:33+02:00
draft: true
title: OSX kernel simple operations
tags: ['mach', 'xnu', 'kernel', 'osx']
---


## Getting current information

Get current `task` with:

    #include <kern/task.h>
    
    task_t cur_task = current_task();

Get current `thread` with:

    #include <kern/thread.h>	

    thread_t cur_thread = current_thread();

## Kernel memory allocation

Use `#include <libkern/OSMalloc.h>` in your source code before all other code below.

First you need to create a tag allocation: 

    OSMallocTag gMallocTag = OSMalloc_Tagalloc("allocation.tag.name", OSMT_DEFAULT);

When you finish you will have to release it via:

    OSMalloc_Tagfree(gMallocTag);

Actual memory allocations:

	void *kernel_new_ptr = OSMalloc((uint32_t)size, gMallocTag); 

Actual memory freeing: 

    OSFree(kernel_new_ptr, (uint32_t)size, gMallocTag);

_The annoying part here is that you must keep the size of your allocations somewhere_

If you are using `IOKit` extension you can use this instead of allocating a malloc tag: 

	void *iokit_new_ptr = IOMalloc(size);

	/* using iokit_new_ptr */

	IOFree(iokit_new_ptr, size);

Or simply call malloc itself:

	void * ptr = _MALLOC(size, 1, M_ZERO);
	
	/* free the ptr */

	_FREE(ptr, M_ZERO);

## Semaphores 

Use `#include <mach/mach_types.h>` in your source code before the code below.

Declare a sempahore: 

     semaphore_port_t sem;

Create a semaphore: 

    if (semaphore_create(current_task(), &sem, SYNC_POLICY_FIFO, 0) != KERN_SUCCESS)
    {
    	/* handle error in creation */
    }

* Note that the scheduling policy (of releases threads) can be changed from `SYNC_POLICY_FIFO` to `SYNC_POLICY_FIXED_PRIORITY`, `SYNC_POLICY_LIFO` and `SYNC_POLICY_ORDERMASK`

Wait on a sempahore: 

	semaphore_wait(sem);

Signal and release one waiting thread: 

	semaphore_signal(sem);

Signal and release all waiting threads: 

    semaphore_signal_all(sem);

Destroy the semaphore: 

    semaphore_destroy(current_task(), &sem);

* Note that all waiting threads are released

## Spinlocks 

## Mutex

Declare the needed lock group attributes, lock group and lock attribute variables:

    lck_grp_t *lock_grp;

Allocate the lock group:

    lock_grp = lck_grp_alloc_init("lock_group_name", LCK_GRP_ATTR_NULL);
	
Declare the lock itself: 

    lck_mtx_t * lock =  lck_mtx_alloc_init(lock_grp, LCK_ATTR_NULL);

Lock the mutex: 

    lck_mtx_lock(lock);

Unlock the mutex:

    lck_mtx_unlock(lock);

Destroy the mutex:

    lck_mtx_destroy(lock);

Free the mutex:

    lck_mtx_free(lock, lock_grp);

Sleep on event: 

    lck_mtx_sleep(lock, LCK_SLEEP_DEFAULT, event, THREAD_INTERRUPTIBLE);

the sleep action can be either:

1. `LCK_SLEEP_DEFAULT` which release the lock while waiting for the event, then reclaim
2. `LCK_SLEEP_UNLOCK` which release the lock and return unheld
3. `LCK_SLEEP_SHARED`  which reclaim the lock in shared mode (RW only)
4. `LCK_SLEEP_EXCLUSIVE` which reclaim the lock in exclusive mode (RW only) 
5. `LCK_SLEEP_SPIN`  which  reclaim the lock in spin mode (mutex only) 

the interrupt action can be either: 

1. `THREAD_UNINT` which is not interruptable
2. `THREAD_INTERRUPTIBLE` which is interruptable but not restable
3. `THREAD_ABORTSAFE` which can be aborted safely

Sleeping with `msleep()`:
{% highlight c++ %}
void *some_data = /* data you are waiting on */ dummyptr;
 
struct timespec ts = {0};
ts.tv_sec   = 0;
ts.tv_nsec  = 10; 

msleep(some_data, lock, 0, "sleep_on_some_data", &ts);
{% endhighlight %}
## Obtain kernel version

include `#include <libkern/version.h>` and use the variables described there:
`version_major, version_minor, version_variant, version_revision, version_stage, version_prerelease_level, ostype, osrelease, osbuilder`

And as a complete string of everything: `version`

## Read from file

{% highlight c++ %}
#include <sys/vnode.h>
vnode_t vnode = NULLVP; 

if (vnode_lookup("/file/path", 0, &vnode, vfs_context_current()))
{
    /* handle the error */
}
else
{
    int err = KERN_SUCCESS;
    offset_t offset = 0;
    user_size_t size = 1024;         
    void *read_buffer = _MALLOC(size, 1, M_ZERO);

    uio_t uio = uio_create(1, offset, UIO_SYSSPACE, UIO_READ);

    if (uio == NULL)
    {
        /* handle the error */
    }
        
        err = uio_addiov(uio, CAST_USER_ADDR_T(read_buffer), size);
        if (err)
        {
           /* handle the error */
        }

        err = VNOP_READ(vnode, uio, 0, vfs_context_current());

        if (err)
        {
            /* handle the error */
        }
        else if (uio_resid(uio))
        {
            /* handle the error */
        }

        uio_free(uio);
        vnode_put(vnode);
    }
{% endhighlight %}
