---
title: "Old macOS kernel debugging with LLDB"
date: 2016-11-04T15:36:58+02:00
---
**Note: This won't work anymore, Apple has pretty much disabled any ability to work with kernel extensions. This is just an old blog post.**
**Kext are no longer supported, use System Extensions and similar approach**


# A quick tutorial for macOX kernel debugging & LLDB

## Start

1. Start by grabbing your favorite LLDB and macOS version
2. Download the current KDK from Apple's developers website
3. Follow the KDK installation guide
4. Configure your VM (or HW) machine
5. Configure VM's boot-args to "debug=0x144 -v" (and if you have 10.12 disable SIP first in recovery mode, then reboot, then sudo nvram boot-args="...")
6. Load your kext and cause a kernel panic by either NULL dereference or direct asm("int3") [which I find the fastest way], and viola!!
7. You now need to open up lldb with the correct kernel (from the KDK) as your executable 

Shell:

    (lldb) kdp-remote [YOUR debugge IP]
    Version: Darwin Kernel Version 16.1.0: Tue Sep 27 21:00:33 PDT 2016; root:xnu-3789.21.1~7/RELEASE_X86_64; UUID=240C9415-166C-300B-B856-B9154FD85130; stext=0xffffff800a600000
    Kernel UUID: 240C9415-166C-300B-B856-B9154FD85130
    Load Address: 0xffffff800a600000
    Kernel slid 0xa400000 in memory.
    Loaded kernel file ./kernel

Now you have your debugger connected, and if you follow up my starting section correctly you know it's damn hard to debug the macOS kernel right?

**Let's go**

## A list of commands

Press help in lldb

    (lldb) help

See the list and cry. This is from 10.12.1 

    abs2nano                                
    addkext                                 
    allproc                                 
    beginusertaskdebugging                  
    btlog_find                              
    calcvmpagehash                          
    disablecore                             
    dumpcallqueue                           
    dumpcrashed_thread_queue                
    dumpobject                              
    dumpthread_terminate_queue              
    findelem                                
    findoldest                              
    findregistryentries                     
    findregistryentry                       
    findregistryprop                        
    fullbt                                  
    fullbtall                               
    getdumpinfo                             
    ifconfig                                
    ifma_showdbg                            
    ifma_trash                              
    ifpref_showdbg                          
    im6o_showdbg                            
    .
    .
    . ( a VERY LONG LIST OF HELPER LLDB FUNCTIONS provided by Apple)
    .
    writemsr64                             
    writephys                              
    xi                                     
    xnudebug                               
    zombproc                               
    zombstacks                             
    zombtasks                              
    zprint                                 
    zstack                                 
    zstack_findelem                        
    zstack_findleak                        
    zstack_inorder                         
    zstack_showzonesbeinglogged            

### abs2nano                                
    convert *mach_absolute_time* units to nano seconds
In *x86_64* world, it does nothing as *mach_absolute_time* is nano second. 

See for yourself, this is the code of *absolutetime_to_nanoseconds*:

    void
    absolutetime_to_nanoseconds(
        uint64_t        abstime,
        uint64_t        *result)
    {
        *result = abstime;
    }

So, not needed. 

BTW, In *ARM* it should be different.


### addkext                                 
    Add kext symbols into lldb.

It should load a kext, and usually it does not...

### allproc                                 
    Walk through the allproc structure and print procinfo for each process structure.

Prints a lovely list of processes and their flags

    Process 0xffffff80111f5e90
        name UserEventAgent
        pid:37     task:0xffffff8018433000   p_stat:2      parent pid: 1
    Cred: euid 0 ruid 0 svuid 0
    Flags: 0x4004
        0x00000004 - process is 64 bit
        0x00004000 - process has called exec
    State: Run
    Process 0xffffff80111f5a18
        name syslogd
        pid:36     task:0xffffff8011362000   p_stat:2      parent pid: 1
    Cred: euid 0 ruid 0 svuid 0
    Flags: 0x4004
        0x00000004 - process is 64 bit
        0x00004000 - process has called exec
    State: Run
    Process 0xffffff80111f55a0
        name launchd
        pid:1      task:0xffffff80109c0aa0   p_stat:2      parent pid: 0
    Cred: euid 0 ruid 0 svuid 0
    Flags: 0x4004
        0x00000004 - process is 64 bit
        0x00004000 - process has called exec
    State: Run
    Process 0xffffff800aeba360
        name kernel_task
        pid:0      task:0xffffff80109c0000   p_stat:2      parent pid: 0
    Cred: euid 0 ruid 0 svuid 0
    Flags: 0x204
        0x00000004 - process is 64 bit
        0x00000200 - system process: no signals, stats, or swap
    State: Run


### beginusertaskdebugging                  
    starts a gdb protocol server that is backed by <task_t> in kernel debugging session.

The coolest feature in the world. Open up a userspace debugging, through gdb remote. 


### fullbt                                  
    Show full backtrace across the interrupt boundary

Grab your RBP via `register read rbp` and put it in fullbt and see your backtrace, or you can use `bt` and get over with it

### fullbtall                               
    Show full backtrace across the interrupt boundary for threads running on all processors

                             
### memstats                               
    (lldb) memstats
    memorystatus_level:         64
    memorystatus_available_pages: 4294967295
    inuse_ptepages_count:         27513
    vm_page_throttled_count:          0
    vm_page_active_count:         77352
    vm_page_inactive_count:      105981
    vm_page_wire_count:          152900
    vm_page_free_count:          119939
    vm_page_purgeable_count:        200
    vm_page_inactive_target:     163932
    vm_page_free_target:           4000
    vm_page_free_reserved:          770


### paniclog                               
Will print the panic log in the debugger as if you have the .panic file. Really useful

### printuserdata                          
read from user space
               
### showalltasks                           
    Routine to print a summary listing of all the tasks
Prints a lovely list of all the tasks in the system 

    task                 vm_map               ipc_space            #acts flags    pid       process             io_policy  wq_state  command
    0xffffff80109c0000   0xffffff800d3216e8   0xffffff8010699780     139            0   0xffffff800aeba360                -1 -1 -1    kernel_task
    0xffffff80109c0aa0   0xffffff8011223360   0xffffff8010699880       7            1   0xffffff80111f55a0                -1 -1 -1    launchd
    0xffffff8018433000   0xffffff8018410d90   0xffffff80183d2880       3 D         37   0xffffff80111f5e90                -1 -1 -1    UserEventAgent
    0xffffff8018451550   0xffffff8018411458   0xffffff80183d2800       2 D         40   0xffffff80111f6308             TQ -1 -1 -1    uninstalld
    0xffffff8018451aa0   0xffffff8018411740   0xffffff80183d27c0       2 D         41   0xffffff80111f4cb0                -1 -1 -1    kextd

The first column `task` is the pointer to `task_t`, the second column `vm_map` is a pointer to `vm_map` and the third is a pointer to `ipc_space`. 


### showallthreads                         
    Display info about all threads in the system

Prints a lovely list of all the threads in the system 

    task                 vm_map               ipc_space            #acts flags    pid       process             io_policy  wq_state  command
    0xffffff80109c0000   0xffffff800d3216e8   0xffffff8010699780     139            0   0xffffff800aeba360                -1 -1 -1    kernel_task
        thread                   thread_id  processor            base   pri    sched_mode      io_policy       state    ast          waitq                            wait_event           wmesg                thread_name
        0xffffff800aece098       0x65       0xffffff800ae8a488   92     92     fixed bound                     WU       L            0xffffff8044267de0               0xffffff800ae87830 <vm_page_free_wanted>
        0xffffff80109d8bb8       0x66       0xffffff800ae8a488   0      0      fixed bound                     RI       L
        0xffffff80109d8720       0x67       0xffffff800ae8a488   95     95     fixed                           WU       L            0xffffff8044267810               0xffffff800a70d8b0 <sched_timeshare_maintenance_continue>                      sched_maintenance_thread
        0xffffff80109d9050       0x68       0xffffff8044b96000   80     80     fixed                           WU       L            0xffffff80442698b0               0xffffff800aece9f0 <osversion>
        0xffffff80109d94e8

Actually it's not very lovely, as the tasks and threads are mixed but you get the point. 

### showinitchild                          
Prints all childs of init process, not really useful outside of boot debugging (and you don't work in Apple right?)

### showioalloc                            
    (lldb) showioalloc
    Instance allocation  = 0x262bf8 =  2442K
    Container allocation = 0x20adef =  2091K
    IOMalloc allocation  = 0x20891bf =  33316K
    Container allocation = 0xc08000 =  12320K
### showpid                                
    Routine to print a summary listing of task corresponding to given pid

Same as `showproc` only with pid
showpipestats                          
showport                               
showportsendrights                     
showprepost                            
showprepostchain                       
### showproc                               
    Routine to print a summary listing of task corresponding to given proc

Grab a `proc_t` pointer from `showproctree` for example, and prints out its info 

    (lldb) showproc 0xffffff8018bbe128
    task                 vm_map               ipc_space            #acts flags    pid       process             io_policy  wq_state  command
    0xffffff8019bf1000   0xffffff8019feb0f8   0xffffff8011c2dec0       2 D        714   0xffffff8018bbe128                -1 -1 -1    iTerm2               -1 -1 -1    launchd   


### showprocfiles                          
    Given a proc_t pointer, display the list of open file descriptors for the referenced process.

Will print a list of all the files in the process (`proc_t` pointer)

    (lldb) showprocfiles 0xffffff80111f55a0
    FD    FILEGLOB           FG_FLAGS   FG_TYPE  FG_DATA            INFO
    ----- ------------------ ---------- -------- ------------------ ----------------------------------------------------------------
    0     0xffffff801123f260 0x00000000 VNODE    0xffffff8011264c98 null
    1     0xffffff801123f2c0 0x00000000 VNODE    0xffffff8011264c98 null
    2     0xffffff801123f320 0x00000010 VNODE    0xffffff80112645d0 console
    3     0xffffff801123f320 0x00000000 VNODE    0xffffff80112645d0 console
    4     0xffffff801123f200 0x00000000 VNODE    0xffffff80112642e8 auditsessions
    5     0xffffff801123f1a0 0x00000000 SOCKET   0xffffff80112d4058
    6     0xffffff801123f080 0x00000000 SOCKET   0xffffff80112d3ca0
    7     0xffffff801123f5c0 0x00000000 SOCKET   0xffffff80112d4410


### showprocfilessummary                   
    Process              Name                 Number of Open Files
    0xffffff801963e250   git                           4
    0xffffff8019638d68   sh                            5
    0xffffff801963a838   git                           4
    0xffffff801963be90   secinitd                      5
    0xffffff801963d960   soagent                       7
    0xffffff80196391e0   bash                          5
    0xffffff80196388f0   useractivityd                 4
    0xffffff801963eb40   nehelper                      6
    0xffffff8019638478   pkd                           3
    0xffffff801963b128   AddressBookSourc              3
    0xffffff801a4dbdd8   com.apple.Perfor              3
    0xffffff801a4db070   syncdefaultsd                 3
    0xffffff801a4db960   IDSKeychainSynci              3
    0xffffff801a4dcb40   CloudKeychainPro              3

Prints a list of open files perh process


### showprocinfo   
    Routine to display name, pid, parent & task for the given proc address                        

Display various info about the process, and has a nice way of printing the process flags in a human format

    (lldb) showprocinfo 0xffffff80111f55a0
    Process 0xffffff80111f55a0
        name launchd
        pid:1      task:0xffffff80109c0aa0   p_stat:2      parent pid: 0
    Cred: euid 0 ruid 0 svuid 0
    Flags: 0x4004
        0x00000004 - process is 64 bit
        0x00004000 - process has called exec
    State: Run
showproclocks                          
showprocsockets                        
### showproctree                           
     Routine to print the processes in the system in a hierarchical tree form. This routine does not print zombie processes.
            If no argument is given, showproctree will print all the processes in the system.
            If pid is specified, showproctree prints all the descendants of the indicated process

Prints all the processes in a nice tree format, and if you give it a pid, just the childs of this process

    PID    PROCESS        POINTER
    ===    =======        =======
    1      launchd        [  0xffffff80111f55a0 ]
    |--712    iTerm2           [  0xffffff8018bbea18 ]
    |  |--875    iTerm2           [  0xffffff80199e94e8 ]
    |  |  |--877    login            [  0xffffff80199ea6c8 ]
    |  |  |  |--881    bash             [  0xffffff80199eab40 ]
    |  |--714    iTerm2           [  0xffffff8018bbe128 ]
    |  |  |--715    login            [  0xffffff8018bbee90 ]
    |  |  |  |--716    bash             [  0xffffff8018bbfbf8 ]
    |  |  |  |  |--846    bash             [  0xffffff80199e75a0 ]
    |  |  |  |  |  |--874    ruby             [  0xffffff80199e8308 ]
    |  |  |  |  |  |  |--24131  bash             [  0xffffff80196391e0 ]
    |  |  |  |  |  |  |  |--24300  git              [  0xffffff801963a838 ]
    |  |  |  |  |  |  |  |  |--24301  sh               [  0xffffff8019638d68 ]
    |  |  |  |  |  |  |  |  |  |--24344  git              [  0xffffff801963e250 ]


### showprocvnodes                         

Display process vnodes

    (lldb) showprocvnodes 0xffffff80111f55a0
    Current Working Directory:
    vnode                usecount  iocount v_data               vtype  parent               mapped cs_version name
    0xffffff8011168c98        162        0 0xffffff806a00b668   VDIR   0x0                  -      -      Macintosh HD

    fd    flags  vnode                usecount  iocount v_data               vtype  parent               mapped cs_version name
    0            0xffffff8011264c98        426        0 0xffffff80111a6d00   VCHR   0xffffff8011169740   -      -      null
    1            0xffffff8011264c98        426        0 0xffffff80111a6d00   VCHR   0xffffff8011169740   -      -      null
    2            0xffffff80112645d0          1        0 0xffffff80111a6700   VCHR   0xffffff8011169740   -      -      console
    3            0xffffff80112645d0          1        0 0xffffff80111a6700   VCHR   0xffffff8011169740   -      -      console
    4            0xffffff80112642e8          1        0 0xffffff80111a6100   VCHR   0xffffff8011169740   -      -      auditsessions
    36           0xffffff80120678b8          2        0 0xffffff886ac82860   VREG   0xffffff8012067aa8   0      0      audit_control
    39           0xffffff80119ac360          2        0 0xffffff886ac7fa40   VREG   0xffffff8012067aa8   0      0      audit_class
### showtaskvm                             
     Display info about the specified task's vm_map


### switchtoact                            
    Switch to different context specified by activation

An annoying name for switch to a different thread. Use `showallthreads` for example, and pick up a thread from there and use it like: 

    switchtoact 0xffffff80181989a8

Now, you can view a different backtrace

### systemlog                              
    Display the kernel's printf ring buffer

Prints out the IOLog and printfs() of your code
                     






