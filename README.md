Project1 :Threads
===
## 11612125 陈新炜
## Task 1: Efficient Alarm Clock
* ### Data structures and functions
  * `struct thread`: add a member `int64_t ticks_count;`in the struct. The struct is in `thread.h`, `ticks_count` will be used to count the rest ticks of a sleeping thread. If `ticks_count` is 0, it means that the thread will be woke up to CPU.
  * `void timer_sleep (int64_t ticks)`: The function is in `timer.c`, add a judge in `timer_sleep`, the thread will be unblocked until `ticks_count` is 0.
* ### Algorithms
  * Check whether the input `ticks` is valid(`ticks > 0`)
  * `ticks_count` will record the sleep time of the thread.
  * Block the thread.
  * Every tick the system will check whether the `ticks_count` is 0. if yes, unblock the thread.
  * Every tick, the `ticks_count` will minus 1.
  <br>
 In this way, the system will record the sleeping time of the thread, and block it until the time is over. During blocking, thread will sleep and will not affect the work of CPU. The thread will unblock only when the `ticks_count` is getting to 0.
  
