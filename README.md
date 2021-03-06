
# Authors:
- Devashi Tandon
- Pratyush Parimal
- Lawrence Candes

# FILES SUBMITTED

- flo-kernel/arch/arm/kernel/calls.S: system call entries.
- flo-kernel/arch/arm/include/asm/unistd.h: declaraing constants for syscall numbers.
- flo-kernel/include/linux/acceleration.h: structures and constants for syscalls.
- flo-kernel/include/linux/mylist.h: structures, interfaces and constants for list operations.
- flo-kernel/kernel/acceleration.c: implementation os all syscalls.
- flo-kernel/kernel/mylist.c: implementation of all list operaations.
- flo-kernel/kernel/Makefile: makefile to build everything.

- acceleration_d/accelerationd.c: daemon that just calls set_acceleration (for part 2)
- acceleration_d/daemon_part3.c: daemon that calls signal (for part 4)

- test/test.c: some tests for event queue.
- test/test_acc.c: main test program that spawns children, creates events, goes to wait, and prints if shakes are detected or not on wakeup.


# TESTS

Setting acceleration values inside the kernel:
---------------------------------------------
We use the daemon 'acceleration_d' for this. It keeps calling 'set_acceleration' and passes the sscaled-up values of x,y,z sensor data to it. The data is used to calculate delta from the previous value, which is then put into a FIFO queue. The queue is used to maintain a SLIDING-WINDOW of max 20 delta-elements. Once it's full the oldest element is deleted before putting in the subsequent new elements.

Creation & deletion of events:
-----------------------------
To do this we call 'test', it creates 5 events in the kernel, and then deletes them. It also tests deletion using invalid event_id and deletion on an empty event-queue.

Forking N processes and creating N events:
-----------------------------------------
We use 'test_acc' for this. It creates N=10 children (this N can be manipulated using command-line args). Each child creates an event using some random dx,dy,dz & frq values, and then waits in the wait-queue. When woken-up, each of them tests whether the condition they're waiting on has been satisfied or not. If not, they go back to sleep, else they check the type of wake-up. If the wake-up was caused due to the actual occurrence of the event, the wait syscall returns 0. Otherwise if they were woken up because their event was destroyed, the syscall returns EINVAL. Based on these values the child can choose to print whether a shake was detected or not.

Sending acceleration values to the kernel and checking if they satisfy events:
-----------------------------------------------------------------------------
For this we use 'daemon_part3'. it keeps calling signal syscall after every 200ms, and sends the x,y,z values and puts the computed delta values in a queue. It also computes the list of deltas for x,y & z. It then iterates the list of events to check if any of them have occurred, upon which it sets their 'condition' as 1.


# REFERENCES

1. http://tuxthink.blogspot.com/2014/07/creating-queue-in-linux-kernel-using.html
2. http://lwn.net/Articles/22913/
