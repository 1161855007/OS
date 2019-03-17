Project1 :Threads
===
## 11612125 陈新炜
## Task 1: Efficient Alarm Clock
* ### Data structures and functions
  * `struct thread`: add a member `int64_t ticks_count;`in the struct. The struct is in *thread.h*, **ticks_count** will be used to count the rest ticks of a sleeping thread. If **ticks_count** is 0, it means that the thread will be woke up to CPU.
  * `void timer_sleep (int64_t ticks)`: The function is in *timer.c*, add a judge in `timer_sleep`, the input **ticks** should be valid.
  * `void thread_check (struct thread *t)`: This function will be written in *thread.c*, and is setted to confirm the status of each thread and is aimed to avoid race condition. When **ticks_count** is 0, it will unblock the blocked thread. Or "wake up the sleeping thread".
* ### Algorithms
  * Check whether the input **ticks** is valid(`ticks > 0`)
  * **ticks_count** will record the sleep time of the thread.
  * Block the thread.
  * Every tick the system will check whether the **ticks_count** is 0. if yes, unblock the thread.
  * Every tick, the **ticks_count** will minus 1.
  <br>
 In this way, the system will record the sleeping time of the thread, and block it until the time is over. During blocking, thread will sleep and will not affect the work of CPU. The thread will unblock only when the **ticks_count** is getting to 0.
* ### Synchronization
  * Race condition will not happen, because for each thread, it has its own value of **ticks_count** as a local variable. when mutiple threads call the `timer_sleep()` simultaneously, it will calculate the sleeping time and wake up it in its own area and will not affect others' status.
  * if ticks is a invalid value, the thread will not sleep successfully, so it can avoid some faults.
* ### Rationale
  * I used to consider that adding the calculation of **ticks_counted** in `timer_sleep()`, but when I turned to write the part of synchronization in this design review, I found that if I do so, race condition will not be avoided. When mutiple threads call the `timer_sleep()` simultaneously, it cannot tell which value is owned by which thread. Therefore, I decide to complete a function called `thread_check` in *thread.c* to make **ticks_count** calculated in each thread's area.
## Task 2: Priority Scheduler
* ### Data structures and functions
  *  `bool compare_priority (const struct list_elem *a, const struct list_elem *b)`: Used for comparing the priority of thread a and thread b, and return a bool value.
  *  `void thread_unblock (struct thread *t)`:change the `list_push_back` to `list_insert_ordered` , this function is in *thread.c*, and `list_insert_ordered` and `list_push_back` are implemented in *list.c* which is in lib/kernel. 
  *  `void thread_yield (void)`: change the `list_push_back` to `list_insert_ordered` , this function is in *thread.c*, and `list_insert_ordered` and `list_push_back` are implemented in *list.c* which is in lib/kernel.
  *  `static void init_thread (struct thread *t, const char *name, int priority)`:change the `list_push_back` to `list_insert_ordered` , this function is in *thread.c*, and `list_insert_ordered` and `list_push_back` are implemented in *list.c* which is in lib/kernel.
  * `struct thread`: add  member `int base_priority;`this integer is used to store the value of priority when the priority donation has not happened. add member `struct lock *waiting;` The lock that the thread is waiting for, if not waiting for any lock, this part can be null. add member `struct list locks;`List of locks that the thread has. add member `struct list waiting_lock;`The list of other threads waiting on locks , they are the potential donor to this thread.
  * `void donate_priority (struct thread *t)` implement the donation of priority.
  * `void back_priority (struct thread *t)` undoing donations when a lock is released.
* ### Algorithms
  *  when a higher priority thread is waiting for acquiring a lock, which is held by a lower priority thread, it should donate its priority to the lower priority thread and record its current base priority. In addition, if this lock is held by other thread, which means that there is a list waiting for the lock, we need to donate the priority recursively.
  *  The thread which has the most donor should be setted as the max priority. Because it is the source user of the lock, and it needs to release the lock as soon as possible.
  *  When there is a list waiting for the lock, we should make the list as a priority queue. Therefore, higher-priority thread will be able to use lock earlier.
  *  During the priority donation, the value of priority for each thread should be updated immediately. If there is a delay of update, the higher-priority thread may not get back his priority and miss the chance to acquire the lock.
  *  During the priority donation, we should make sure that the value of priority after donated is lager than the current priority. If not, do not execute the instruction.
  *  When the lock is released, beneficiary thread and donor thread should update their priority to base priority immediately.
  *  
* ### Synchronization
* ### Rationale


  

