Project1 :Threads
===
## 11612125 陈新炜
## Task 1: Efficient Alarm Clock
* ### Data structures and functions
  * `struct thread`: add a member `int64_t ticks_count;`in the struct. The struct is in `thread.h`, `ticks_count` will be used to count the rest ticks of a sleeping thread. If `ticks_count` is 0, it means that the thread will be woke up to CPU.
  * `void timer_sleep (int64_t ticks)`: The function is in `timer.c`, add a judge in `timer_sleep`, the input `ticks` should be valid.
  * `void thread_check (struct thread *t)`: This function will be written in `thread.c`, and is setted to confirm the status of each thread and is aimed to avoid race condition. When `ticks_blocked` is 0, it will unblock the blocked thread. Or "wake up the sleeping thread".
* ### Algorithms
  * Check whether the input `ticks` is valid(`ticks > 0`)
  * `ticks_count` will record the sleep time of the thread.
  * Block the thread.
  * Every tick the system will check whether the `ticks_count` is 0. if yes, unblock the thread.
  * Every tick, the `ticks_count` will minus 1.
  <br>
 In this way, the system will record the sleeping time of the thread, and block it until the time is over. During blocking, thread will sleep and will not affect the work of CPU. The thread will unblock only when the `ticks_count` is getting to 0.
* ### Synchronization
  * Race condition will not happen, because for each thread, it has its own value of `ticks_blocked` as a local variable. when mutiple threads call the `timer_sleep()` simultaneously, it will calculate the sleeping time and wake up it in its own area and will not affect others' status.
  * if ticks is a invalid value, the thread will not sleep successfully, so it can avoid some faults.
* ### Rationale
  * I used to consider that adding the calculation of `ticks_blocked` in `timer_sleep()`, but when I turned to write the part of synchronization in this design review, I found that if I do so, race condition will not be avoided. When mutiple threads call the `timer_sleep()` simultaneously, it cannot tell which value is owned by which thread. Therefore, I decide to complete a function called `thread_check` in `thread.c` to make `ticks_blocked` calculated in each thread's area.
## Task 2: Priority Scheduler
* ### Data structures and functions
  *  `bool thread_cmp_priority (const struct list_elem *a, const struct list_elem *b)`: Used for comparing the priority of thread a and thread b, and return a bool value.
  *  `void thread_unblock (struct thread *t)`:change the `list_push_back` to `list_insert_ordered` , this function is in `thread.c`, and `list_insert_ordered` and `list_push_back` are implemented in list.c which is in lib/kernel. 
  *  `void thread_yield (void)`: change the `list_push_back` to `list_insert_ordered` , this function is in `thread.c`, and `list_insert_ordered` and `list_push_back` are implemented in list.c which is in lib/kernel.
  *  `static void init_thread (struct thread *t, const char *name, int priority)`:change the `list_push_back` to `list_insert_ordered` , this function is in `thread.c`, and `list_insert_ordered` and `list_push_back` are implemented in list.c which is in lib/kernel.
  
