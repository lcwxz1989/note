# _

## Concurrency: An Introduction

## Locks

### Spin Locks

#### Evaluation

1. mutual exclusion
   yes
2. fairness
   not guarantees
3. performance

### compare-and-swap

better than tas in lock-free synchro- nization topic
identical behavior in spin lock

### Load-Linked and Store-Conditional

The key difference comes with the store-conditional, which only succeedsif no intervening store to the address has taken place.
Suppose, if processor A put variable `a ptr` to `register R`; Processor B put variable `b ptr` to register R again. When A update a will failed because of R have store variable `b ptr`. Deeper, `update primitive` means get ptr and set it(in other words, it include two actions), so if get ptr failed, set will failed.

### Fetch-And-Add

one important difference with this solution versus our previous attempts: it ensures progress for all threads

### HOW TO AVOID SPINNING -- OS

## Condition Variables

### Definition and Routines

condition variable: an explicit queue that threads can put themselves on when some state of execution (i.e., some condition) is not as desired

primitives:
   wait: First step is release the lock and put the calling thread to sleep (atomically);
         Sencond step iswhen the thread wakes up (after some other thread has signaled it), it must re-acquire the lock before returning to the caller.

      ```c++
         void thr_join() { 
            Pthread_mutex_lock(&m); 
            while (done == 0)
               Pthread_cond_wait(&c, &m);
            Pthread_mutex_unlock(&m);
         }
      ```

   signal:

      ```c++
      void thr_exit() { 
         Pthread_mutex_lock(&m); 
         done = 1; 
         Pthread_cond_signal(&c); 
         Pthread_mutex_unlock(&m);
      }
      ```
## The Producer/Consumer(Bounded Buffer)Problem
