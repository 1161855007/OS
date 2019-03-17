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
  *  If priority changed when releasing the lock, it is permitted to change the order in waiting queue.
  *  when releasing the lock, we should check the current priority and base priority of the thread, and it is possilble that the thread is donated again when releasing beacuse a thread may hold more than one lock.
  *  In the list of waiters, front threads should have higher priority, and higher priority thread can be inserted into the queue. Therefore, it make sure that highest priority waiting thread can acquire lock first. 
  *  All relative information of each thread must update data as soon as possible.
  *  At first, here is a simple case:
     *   When a thread holds a lock right now, the thread in the waiting list should check that whether the priority is higher than itself.
     *   if the waiting thread has a higher priority than lock holder, the waiting thread should begin the priority donation.
     *   we can set the lock holder the same priority to the waiting thread, so the priority is the real priority for this lock holder. If not do so, some medium thread may execute earlier than the holder, and it cause that the waiting thread do not enjoy its right of priority. Because in fact, the medium thread execute before the higher waiting thread. It is a wrong order.
     *   After the lock thread release, it will set back its priority immediately. And the waiting higher priority thread can acquire the lock successfully then.
  *  Now, I will try to give a more complex situation:
     *  If the lock holder has more than one lock, we should decide the donate order to this lock.
     *  As the situation that has one lock, we should confirm whether there is donor to this thread.
     *  We should set the lock holder as the same priority to the highset waiting thread's priority, if the lock that the highest priority thread want has released, we should change the lock holder's priority to the second highest thread's priority. And the rest rule, is as the above, if it holds more than two lock.
     *  I guess it is possible to have a condition that there are more than one thread holds more than one locks. To solve this problem, we should judge who is real higher priority thread. I have two try: 1.) We can add all the priority together to decide who has a higher priority. 2.) We can choose the holder which has more donor a higher priority. I prefer the second one, but do not has some solid reason. Maybe the add will cause a overflow is one. It is difficult to decide which one as a higher priority thread, I may test more samples to find the final way.
  * There is a possible special case:
     * when the priority has a donation chain, we should set the lock holder's priority to the highest priority in chain. So it is possilble to find that the next thread has a lower priority that the lock holder. It is nothing and we just need to continue our priority chain. 
  * To sum up, priority scheduler is to implement the queue that base on priority of each thread. Rule of priority donation is essentially to find the real priority of each thread. Only with priority donation, thread can acquire lock in the right order.
    
* ### Synchronization
  * Beacuse the system is mutiple thread, many function may be used at the same time. 
  * `back_priority` will not be affacted, because it has the argument **thread t**. When there is mutiple thread that need to come back to their base priority, they can update their priority respectively.
  * `donate_priority`will not be affacted, because it has the argument **thread t**. we can set the new priority of the thread t without other threads influence.
  * `init_thread ` will not be affacted, because it also has the argument to tell which thread is to initialize.
  * `thread_set_priority` is not good for avoid race condition, we should change it to locate which thread is setting priority.
* ### Rationale
  * To be honest, this part is too difficult and I haven't do some real test to demonstrate that my design is good enough. By now, I take some simple cases into my consideration and it is doomed not thorough enough at all. The rule of priority donation is rational because we follow the real priority to order the thread in queue, no matter how many thread is in priority donation chain; no matter how many locks one holder has.
## Task 3: Multi-level Feedback Queue Scheduler (MLFQS)
* ### Data structures and functions
  * `void thread_set_nice (int nice UNUSED)`: the funciton in *thread.c* which hasn't been implemented.
  * `int thread_get_nice (void)`: the funciton in *thread.c* which hasn't been implemented.
  * `int thread_get_load_avg (void)`: the funciton in *thread.c* which hasn't been implemented.
  * `int thread_get_recent_cpu (void)`: the funciton in *thread.c* which hasn't been implemented.
  * `struct thread`: add member `int nice;`add member `int recent_cpu;`
* ### Algorithms
  * Scheduler has 64 priorities and thus 64 ready queues, numbered 0 ( PRI_MIN ) through 63 ( PRI_MAX ) ,and lower numbers correspond to lower priorities.
  * Thread priority is calculated initially at thread initialization. It is also recalculated once every fourth clock tick, for every thread.
  * priority = PRI_MAX − (recent_cpu/4) − (nice×2)
  * recent_cpu = (2 × load_avg)/(2 × load_avg + 1) × recent_cpu + nice
  * load_avg = (59/60)× load_avg + (1/60) × ready_threads
* ### Synchronization
  * each thread in queue has different priority and they can be told from each other, so I think there won't be problem in this part.
* ### Rationale
  *  In this part, we use the given formula to implement MLFQS. The formula is designed well and is based on float calculation. I haven't find any hidden problem in the given design. I think the concept of MLFQS is excellent and rational.

## Design Document Additional Questions
 |timer ticks | R(A) | R(B) | R(C) | P(A) | P(B) | P(C)| thread to run |
 | -------- | -----: | :----: | :----: | :----: | :----: | :----: | :----: | 
 | 香蕉 | $1 | 5 | 
 | 苹果 | $1 | 6 | 
 | 草莓 | $1 | 7 |

 

  

