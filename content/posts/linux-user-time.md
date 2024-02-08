---
title: "Linux User Time"
date: 2017-04-18T19:50:53+02:00
draft: true
tags: ['performance']
---


Say we want to measure the number of **user space** instructions a process is spending at a function. There are number of ways to do so: `gettimeofday`, `times`, `clock` and `getrusage`, but they give you time and not CPU cycles and not very accurate. 

Then there is this hidden function: `perf_event_open()`. 

As the _...long..._ [man](http://man7.org/linux/man-pages/man2/perf_event_open.2.html) page states:  "A call to `perf_event_open()` creates a file descriptor that allows  measuring performance information.  Each file descriptor corresponds to one event that is measured; these can be grouped together to measure multiple events simultaneously."

An example from the man page:

{{< highlight cpp >}}

#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <sys/ioctl.h>
#include <linux/perf_event.h>
#include <asm/unistd.h>

long perf_event_open(struct perf_event_attr *hw_event, pid_t pid,
            int cpu, int group_fd, unsigned long flags)
{
    int ret = syscall(__NR_perf_event_open, hw_event, pid, cpu, group_fd, flags);
    return ret;
}

int main(int argc, char **argv)
{
    struct perf_event_attr pe;
    long long count;
    int fd;

    memset(&pe, 0, sizeof(struct perf_event_attr));
    
    // initialize the perf_event_attr, exclude kernel and count CPU instructions
    pe.type = PERF_TYPE_HARDWARE;
    pe.size = sizeof(struct perf_event_attr);
    pe.config = PERF_COUNT_HW_INSTRUCTIONS;
    pe.disabled = 1;
    pe.exclude_kernel = 1;
    pe.exclude_hv = 1;

    // open our performance counter
    fd = perf_event_open(&pe, 0, -1, -1, 0);
    if (fd == -1) {
       fprintf(stderr, "Error opening: %llx\n", pe.config);
       exit(EXIT_FAILURE);
    }

    // start the counter
    ioctl(fd, PERF_EVENT_IOC_RESET, 0);
    ioctl(fd, PERF_EVENT_IOC_ENABLE, 0);

    // measure the number of instructions in the printf 
    printf("Measuring instruction count for this printf\n");

    // also measure this but, who cares :)
    ioctl(fd, PERF_EVENT_IOC_DISABLE, 0);

    // read the number of CPU insntructions
    read(fd, &count, sizeof(long long));
    printf("Used %lld instructions\n", count);

    close(fd);
}

{{< / highlight >}}
