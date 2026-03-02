

# Summary of the main aspects of thread usage for Distributed Systems
+ **Felix Garcia Carballeira and Alejandro Calderon Mateos**
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

* Lightweight processes that are (1) independent, (2) competing, and (3) cooperating:
  * [1. What threads are and how they are created](#1--what-threads-are-and-how-they-are-created)
  * [2. What **mutexes** are and when they are used: two or more threads share a variable and at least one modifies it non-atomically](#2--what-mutexes-are-and-how-they-are-used-for-when-1-two-or-more-threads-2-share-a-variable-3-at-least-one-modifies-the-variable-4-and-it-is-done-in-a-non-atomic-way)
  * [3. What **conditions** are and when to use them: the execution of one thread waits for the execution of another thread's code](#3--what-are-conditions-and-when-to-use-them-make-the-execution-of-a-thread-wait-until-code-from-another-thread-is-executed)
* Request server:
  * [4. Thread-on-demand](#4--thread-based-on-demand-request-server)
  * [5. Thread pool](#5--thread-pool-based-request-server)
* [Additional information](#6--additional-information)


## 1.- <ins>What threads are and how they are created</ins>

As an example, this program has:
* A main function that creates 5 threads and waits for all threads to finish.
* Each thread prints “Hello world from thread &lt;thread id&gt;!\n”

### ths_creatjoin.c
  ```c
  #include <stdlib.h>
  #include <stdio.h>
  #include <pthread.h>
  
  #define NUM_THREADS     5

  pthread_t threads[NUM_THREADS];

  // The th_function function has the code that the threads will execute
  void *th_function ( void *arg )
  {
      printf("Hello world from thread #%ld!\n", (long)arg) ;
      pthread_exit(NULL) ;
  }

  int main ( int argc, char *argv[] )
  {
      // Create NUM_THREADS threads with function th_function and argument the value of t
      for (int t=0; t<NUM_THREADS; t++)
      {
           int rc = pthread_create(&(threads[t]), NULL, th_function, (void *)(long)t) ;
           if (rc) {
               printf("ERROR from pthread_create(): %d\n", rc) ;
               exit(-1) ;
           }
    }
    
    // Wait for NUM_THREADS threads to finish
    for (int t=0; t<NUM_THREADS; t++)
    {
         int rc = pthread_join(threads[t], NULL) ;
         if (rc) {
             printf("ERROR from pthread_join(): %d\n", rc) ;
             exit(-1); 
         }
    }

    // pthread_exit(NULL); // main is the main thread
    return 0;
  }
  ```

To compile and run, please use:
```bash
gcc -Wall -g -o ths_creatjoin ths_creatjoin.c -lpthread
./ths_creatjoin
```

Reminders
* <details>
  <summary>Do not make assumptions about the order of code execution... (click)</summary>
  
  * Do not rely on correct execution of a particular order, but rather on any possible order.
  </details>
* <details>
  <summary>A single parameter is passed to the thread... (click)</summary>
  
  * The API was designed to pass a single parameter that is a pointer to the memory address of a C structure (struct) in which all the thread's working arguments are stored.
  * (1/2) Before creating the thread, you must reserve memory and fill in the working values:
     ```
       th_args = (struct thread_arguments *)malloc(sizeof(struct thread_arguments)) ;
       if (NULL == th_args) {
           printf("ERROR from malloc\n") ;
           exit(-1) ;
           }

           th_args->arg1 = 'a' ;
           th_args->arg2 = 33 ;
           th_args->arg3 = j ;
           ...
           int rc = pthread_create(&(threads[t]), NULL, th_function, (void *)th_args);
           if (rc)
           {
               printf("ERROR from pthread_create(): %d\n", rc) ;
               exit(-1) ;
           }
      ...
  * (2/2) This structure is used in the thread, and the associated memory is freed before termination:
     ```
     void *th_function ( void *arg )
     {
        struct thread_arguments *th_args = (struct thread_arguments *) arg ;
        
         ...
        printf("Hello world from thread #%d!\n", th_args->arg3);
        ...

        free(th_args);
        th_args = NULL;
        pthread_exit(NULL);
     }
     ```
</details>

**Supplementary videos**:
  * <a href="https://www.youtube.com/watch?v=n5qrEotEWfI">Review of thread concepts</a>
  * <a href="https://www.youtube.com/watch?v=akf9UG7Z5Go">Main POSIX thread calls with examples</a>



## 2.- <ins>What **mutexes** are and how they are used: for when (1) two or more threads (2) share a variable (3) at least one modifies the variable (4) and it is done in a non-atomic way</ins>

When (1) two or more threads (2) share a variable (3) at least one modifies the variable (4) and it is done in a non-atomic way, then the problem called “race condition” arises.
The area of code where each thread accesses the shared variable (either to query or modify it) is called the “critical section.”

As an example, this program solves a race condition between two threads by using a mutex called “mutex_1.”

### race_sol.c
  ```c
  #include <stdlib.h>
  #include <stdio.h>
  #include <unistd.h>
  #include <pthread.h>

  long acc ;
  pthread_mutex_t mutex_1; // MUTEX

  void *th_main_1 ( void *arg )
  {
     register int a1 ;

     pthread_mutex_lock(&mutex_1) ; // LOCK
     for (int j=0; j<2; j++)
     {
         a1  = acc ;
             printf("main_1 sees acc=%d...\n", a1) ;
         a1  = a1 + 1 ;
         acc = a1 ;
             printf("main_1 updates acc=%d...\n", a1) ;
         sleep(1) ;
     }
     pthread_mutex_unlock(&mutex_1) ; // UNLOCK

     pthread_exit(NULL); 
  }

  void *th_main_2 ( void *arg )
  {
     register int a2;

     pthread_mutex_lock(&mutex_1); // LOCK
     for (int j=0; j<2; j++)
     {
         a2  = acc ;
             printf("main_2 sees acc=%d...\n", a2) ;
         a2  = a2 - 1 ;
         acc = a2 ;
             printf("main_2 updates acc=%d...\n", a2) ;
	     sleep(1);
     }
     pthread_mutex_unlock(&mutex_1); // UNLOCK

     pthread_exit(NULL);
   }
  
   int main ( int argc, char *argv[] )
   {
      int rc ;
      pthread_t threads[2];

      // Initialize...
      acc = 0 ;
      pthread_mutex_init(&mutex_1, NULL) ; // INIT-MUTEX

      // Create threads...
      rc = pthread_create(&(threads[0]), NULL, th_main_1, NULL);
      if (rc) {
          printf("ERROR from pthread_create(): %d\n", rc);
          exit(-1);
      }
    
      rc = pthread_create(&(threads[1]), NULL, th_main_2, NULL);
      if (rc) {
          printf("ERROR from pthread_create(): %d\n", rc);
          exit(-1);
      }
    
      // Join threads...
      rc = pthread_join(threads[0], NULL);
      if (rc) {
          printf("ERROR from pthread_join(): %d\n", rc) ;
          exit(-1) ;
      }
      rc = pthread_join(threads[1], NULL) ;
      if (rc) {
          printf("ERROR from pthread_join(): %d\n", rc);
          exit(-1);
      }

      printf(" >>>>> acc = %ld\n", acc);

      // Destroy...
      pthread_mutex_destroy(&mutex_1); // DESTROY-MUTEX

      pthread_exit(NULL);
   }
  ```

To compile and execute, use:
```bash
  gcc -Wall -g -o race_sol race_sol.c -lpthread
  ./race_sol
```

Reminders
* <details>
  <summary>Be careful with deadlocks with two or more mutexes... (click)</summary>
  
  * If two or more mutexes overlap, follow the same order of mutex lock requests to avoid deadlocks (one thread waiting for a mutex held by another and vice versa).
  ```c
  #include <stdlib.h>
  #include <stdio.h>
  #include <unistd.h>
  #include <pthread.h>

  pthread_mutex_t mutex_1;
  pthread_mutex_t mutex_2;

  void *th_main_1 ( void *arg )
  {
      pthread_mutex_lock(&mutex_1) ;
      sleep(1);
      pthread_mutex_lock(&mutex_2);

      printf("Hello from main_1...\n");

      pthread_mutex_unlock(&mutex_2);
      pthread_mutex_unlock(&mutex_1);

      pthread_exit (NULL);
  }

  void *th_main_2 ( void *arg )
  {
      pthread_mutex_lock(&mutex_1); // OK
      sleep(1);
      pthread_mutex_lock(&mutex_2); // OK

      printf("Hello from main_2...\n");
      sleep(1);

      pthread_mutex_unlock(&mutex_2);
      pthread_mutex_unlock(&mutex_1);
  
      pthread_exit(NULL);
  }

  int main ( int argc, char *argv[] )
  {
      int rc;
      pthread_t threads [2];

      // Initialize mutexes...
      pthread_mutex_init(&mutex_1, NULL);
      pthread_mutex_init(&mutex_2, NULL);
    
      // Create threads...
      rc = pthread_create(&(threads[0]), NULL, th_main_1, NULL) ;
      if (rc) {
          printf("ERROR from pthread_create(): %d\n", rc) ;
          exit(-1) ;
      }
    
      rc = pthread_create(&(threads[1]), NULL, th_main_2, NULL);
      if (rc) {
          printf("ERROR from pthread_create(): %d\n", rc);
          exit(-1);
      }

      // Join threads...
      for (int t=0; t<2; t++)
      {
           rc = pthread_join(threads[t], NULL) ;
           if (rc) {
               printf("ERROR from pthread_join(): %d\n", rc) ;
               exit(-1) ;
           }
      }

      // Destroy mutexes...
      pthread_mutex_destroy(&mutex_1);
      pthread_mutex_destroy(&mutex_2);

      pthread_exit(NULL);
  }
  ```

  To compile and run, use:
  ```bash
  gcc -Wall -g -o interlock_sol interlock_sol.c -lpthread
  ./interlock_sol
  ```
  </details>

**Supplementary videos**:
  * <a href="https://www.youtube.com/watch?v=PxjgVYgpGkk&t=471s">Race conditions</a>
  * <a href="https://www.youtube.com/watch?v=PxjgVYgpGkk&t=924s">Interlock conditions</a>


## 3.- <ins>What are conditions and when to use them: make the execution of a thread wait until code from another thread is executed</ins>

As an example, this program synchronizes the main thread and the threads created with pthread_create so that “main” waits until the thread has been created and the arguments have also been copied.

### sync_child_mnc_sol.c
  ```c
  #include <stdlib.h>
  #include <stdio.h>
  #include <pthread.h>
  
  #define NUM_THREADS  5

    /// variable + mutex + condition /////
    int             is_copied = 0;   // (1/3) Boolean indicating "has been copied"
    pthread_mutex_t mutex_1;         // (2/3) mutex that protects the global variable "is_copied":
                                     //       * by 2 or more threads,
                                     //       * the variable is shared 
                                     //       * modified by one
                                     //       * and the modification is not atomic.
    
    pthread_cond_t cond_cp;         // (3/3) condition to wait if the Boolean asks to wait
    /////////////////////////////////
  
   void *th_function ( void *arg )
   {
      int *p_thid ;

      p_thid = (int *)arg ;

      /// Notifies that it has been copied ///
      pthread_mutex_lock(&mutex_1) ;
      is_copied = 1 ;
    
      pthread_cond_signal(&cond_cp);
      pthread_mutex_unlock(&mutex_1);
      ///////////////////////////////

      printf("Hello world from thread #%d!\n", *p_thid);
      pthread_exit(NULL);
  }

  int main ( int argc, char *argv[] )
  {
      int t, rc ;
      pthread_t threads[NUM_THREADS];

      // Initialize
      pthread_mutex_init(&mutex_1, NULL) ;
      pthread_cond_init(&cond_cp, NULL) ;
    
      // Create threads...
      for (t=0; t<NUM_THREADS; t++)
      {
           rc = pthread_create(&(threads[t]), NULL, th_function, (void *)&t);
           if (rc) {
               printf("ERROR from pthread_create(): %d\n", rc) ;
               exit(-1) ;
           }

           /// If not copied, wait for copy notification ///
           pthread_mutex_lock(&mutex_1) ;
           while (0 == is_copied) {
                  pthread_cond_wait(&cond_cp, &mutex_1) ;
           }
           is_copied = 0;
           pthread_mutex_unlock(&mutex_1) ;
           ////////////////////////////////////////////
      }
    
      // Wait for threads...
      for (t=0; t<NUM_THREADS; t++)
      {
           rc = pthread_join(threads[t], NULL) ;
           if (rc) {
               printf("ERROR from pthread_join(): %d\n", rc) ;
               exit(-1) ;
           }
      }
    
      // Destroy...
      pthread_mutex_destroy(&mutex_1) ;
      pthread_cond_destroy(&cond_cp) ;

      pthread_exit (NULL);
    }
   ```

To compile and run, use:
```bash
  gcc -Wall -g -o sync_child_mnc_sol sync_child_mnc_sol.c -lpthread
  ./sync_child_mnc_sol
```

Reminders
* <details>
  <summary>signal vs broadcast... (click)</summary>

  * To wake up only one thread waiting in the condition:
    ```
    pthread_cond_signal(&cond_cp) ;
    ```
  * To wake up all threads waiting in the condition:
    ```
    pthread_cond_broadcast(&cond_cp) ;
    ```
  </details>

**Supplementary video**:
  * <a href="https://www.youtube.com/watch?v=EupaagvNpR0&t=807s">How mutexes and conditions work</a>


## 4.- <ins>Thread-based on-demand request server</ins>

As an example, this program simulates a thread-based on-demand request server.
In order for the thread on the server that handles the client's request to be created as soon as possible, the service function waits until the child thread has been created and that thread has copied the parameters.

### threads_ondemand.c
  ```c
  #include <stdio.h>
  #include <stdlib.h>
  
  #include <string.h>
  #include <sys/time.h>
  #include <pthread.h>
  #include <unistd.h>

  // Requests //////////////////////////////

   struct request {
      long id;
      int type;
      /* ... */
   };
   typedef struct request request_t;

   static long petid = 0;

   void receive_request (request_t * p)
   {
      fprintf(stderr, "Receiving request\n");

      /* I/O time simulation */
      int delay = rand() % 10;
      sleep(delay);

      p->id = petid++;
     
      fprintf(stderr, "Request %ld received after %d seconds\n", p->id, delay);
   }
  
   void respond_request (request_t * p)
   {
      fprintf(stderr, "Sending request %ld\n", p->id);

      /* Simulate processing time */
      int delay1 = rand() % 5;
      sleep(delay1);
    
      /* Simulate I/O time */
      int delay2 = rand() % 10;
      sleep(delay2);

      fprintf(stderr, "Request %ld sent after %d seconds\n",  p->id, delay1 + delay2);
   }
  
   // Threads on demand //////////////////////

    pthread_mutex_t mutex;
    pthread_cond_t  copied;
    int             is_copied;

    const int MAX_REQUESTS = 5;

    void * service ( void * p )
    {
        petition_t  pet;
        
        // copy parameters...
        memmove(&pet,(petition_t*)p, sizeof(petition_t));

        // signal data is copied
        pthread_mutex_lock(&mutex) ;
        is_copied = 1 ;
        pthread_cond_signal(&copied) ;
        pthread_mutex_unlock (&mutex);

        // process and response
        fprintf(stderr, "Starting service\n");
        respond_request(&pet);
        fprintf(stderr, "Ending service\n");
        
        pthread_exit(0);
        return NULL;
    }

    void receiver ( void )
    {
       int i;
       request_t  p;
       pthread_t   th_child[MAX_REQUESTS];

       // for each request, a new thread...
       for (i=0; i<MAX_REQUESTS; i++)
       {
            // receive request and new thread treat it
            receive_request(&p);
            pthread_create(&(th_child[i]), NULL, service, &p);

            // wait data is copied
            pthread_mutex_lock(&mutex) ;
            while (!is_copied) {
                   pthread_cond_wait(&copied, &mutex) ;
            }
            is_copied = 0 ;
            pthread_mutex_unlock (&mutex) ;
       }

       // wait for each thread ends
       for (i=0; i<MAX_REQUESTS; i++) {
            pthread_join(th_child[i], NULL) ;
       }
    }

    int main ( int argc, char *argv [])
    {
        struct timeval timenow;

        // t1
        gettimeofday(&timenow, NULL);
        long t1 = (long)timenow.tv_sec * 1000 + (long)timenow.tv_usec / 1000;

        // Request dispatcher...
        receiver();
      
        // t2
        gettimeofday(&timenow, NULL);
        long t2 = (long)timenow.tv_sec * 1000 + (long)timenow.tv_usec / 1000 ;

        // print t2-t1...
        printf("Total time: %lf\n", (t2-t1)/1000.0);
        return 0;
    }
   ```

To compile and run, use:
```bash
gcc -Wall -g -o threads_ondemand  threads_ondemand.c -lpthread
./threads_ondemand
```

**Supplementary video**:
  * <a href="https://www.youtube.com/watch?v=nDyYrpFYG-4&t=551s">Example of a thread-based request server</a>



## 5.- <ins>Thread pool-based request server</ins>

As an example, this program simulates a thread pool-based request server.

### threads_pool.c
  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <string.h>
  #include <sys/time.h>
  #include <time.h>
  #include <pthread.h>
  #include <unistd.h>

  // Requests //////////////////////////////
  struct request {
      long id;
      int type;
      /* ... */
   };
   typedef struct request request_t;
  
   static long petid = 0;

   void receive_request (request_t * p)
   {
      fprintf(stderr, "Receiving request\n");

       /* I/O time simulation */
       int delay = rand() % 10;
       sleep(delay);
     
       p->id = petid++;

       fprintf(stderr, "Request %ld received after %d seconds\n", p->id, delay);
   }

   void respond_request (request_t * p)
   {
       fprintf(stderr, "Sending request %ld\n", p->id);
    
       /* Simulate processing time */
       int delay1 = rand() % 5;
       sleep(delay1);
    
       /* I/O time simulation */
       int delay2 = rand() % 10;
       sleep(delay2);

       fprintf(stderr, "Request %ld sent after %d seconds\n",  p->id, delay1 + delay2);
   }

   // Thread pool //////////// ///////////////

   #define MAX_BUFFER 128
   request_t  buffer[MAX_BUFFER];

   int  n_elements  = 0;
   int  has_started = 0;
   int  end = 0;
  
   pthread_mutex_t mutex;
   pthread_cond_t not_full;
   pthread_cond_t not_empty;
   pthread_cond_t started;
   pthread_cond_t stopped;

   const int MAX_REQUESTS = 5;
   const int MAX_SERVICE = 5;
  
   void * receiver ( void * param )
   {
         request_t p;
         int i, receiver_pos = 0;

         for (i=0; i<MAX_REQUESTS; i++)
         {
              receive_request(&p);
              fprintf(stderr, "receiver: receiving request\n");
            
              // lock when not full...
                 pthread_mutex_lock(&mutex);
                 while (n_elements == MAX_BUFFER) {
                     pthread_cond_wait(&not_full, &mutex);
                 }
            
              // inserting element into the buffer
                 buffer[receiver_pos ] = p;
                 receiver_pos = (receiver_pos +1) % MAX_BUFFER;
                 n_elements++;

              // signal not empty...
                 pthread_cond_signal(&not_empty);
                 pthread_mutex_unlock(&mutex);
         }

         fprintf(stderr, "receiver: finishing\n");

         // signal end
         pthread_mutex_lock(&mutex);
         fin=1;
         pthread_cond_broadcast(&no_empty);
         pthread_mutex_unlock(&mutex);

         fprintf(stderr, "receiver: Finished\n");
         pthread_exit(0);
         return NULL;
   }

   void * service ( void * param )
   {
        request_t p;
        int service_pos = 0;

        // signal initialize...
        pthread_mutex_lock(&mutex);
        has_started = 1;
        pthread_cond_signal(&started);
        pthread_mutex_unlock(&mutex);

        for (;;)
        {
          // lock when not empty and not ended...
             pthread_mutex_lock(&mutex);
             while (n_elements == 0)
             {
                  if (end==1) {
                       fprintf(stderr,“service: ending\n”);
                       pthread_cond_signal(&stopped);
                       pthread_mutex_unlock(&mutex);
                       pthread_exit(0);
                  }

                  pthread_cond_wait(&not_empty, &mutex);
             } // while

          // removing element from buffer...
             p = buffer[service_position];
             service_pos = (service_pos + 1) % MAX_BUFFER;
             n_elements--;

          // signal not full...
             pthread_cond_signal(&not_full);
             pthread_mutex_unlock(&mutex);

          // process and response...
             fprintf(stderr, “service: serving position %d\n”, pos_service);
             respond_request(&p);
        }

        pthread_exit(0);
        return NULL;
   }

   int main ( int argc, char *argv[] )
   {
      struct timeval timenow;
      long t1, t2;
      pthread_t thr;
      pthread_t ths[MAX_SERVICE];

   // initialize
      pthread_mutex_init(&mutex,    NULL);
      pthread_cond_init(&not_full, NULL);
      pthread_cond_init(&not_empty, NULL);
      pthread_cond_init(&started, NULL);
      pthread_cond_init(&stopped, NULL);
      
   // create threads
      for (int i=0;i<MAX_SERVICE;i++)
      {
            pthread_create(&ths[i], NULL, service, NULL);
            
         // wait thread is started
            pthread_mutex_lock(&mutex) ;
            while (!has_started) {
                   pthread_cond_wait(&started, &mutex) ;
            }
            has_started = 0 ;
            pthread_mutex_unlock(&mutex) ;
      }
      
   // t1
      gettimeofday(&timenow, NULL);
      t1 = (long)timenow.tv_sec * 1000 + (long)timenow.tv_usec / 1000;

        // receiver...
           pthread_create(&thr, NULL, receiver, NULL);
            
        // wait thread is started
           pthread_mutex_lock(&mutex) ;
           while ( (!end) || (n_elements > 0) ) {
                    pthread_cond_wait(&stopped, &mutex) ;
           }
           pthread_mutex_unlock(&mutex) ;

        // end
           pthread_join (thr, NULL);
           for (int i=0; i<MAX_SERVICE; i++) {
                pthread_join(ths[i], NULL);
           }
      
   // t2
      gettimeofday(&timenow, NULL);
      t2 = (long)timenow.tv_sec * 1000 + (long)timenow.tv_usec / 1000;

      pthread_mutex_destroy(&mutex);
      pthread_cond_destroy(&no_full);
      pthread_cond_destroy(&not_empty);
      pthread_cond_destroy(&started);
      pthread_cond_destroy(&stopped);
      
   // print t2-t1...
      printf("Total time: %lf\n", (t2-t1)/1000.0);
      return 0;
  }
  ```

To compile and run, use:
```bash
gcc -Wall -g -o threads_pool  threads_pool.c -lpthread
./threads_pool
```

**Supplementary video**:
  * <a href="https://www.youtube.com/watch?v=nDyYrpFYG-4&t=940s">Example of a request server based on pre-created threads</a>


<br>

## 6.- Additional information
  * [Examples for Operating Systems (github)](https://github.com/acaldero/uc3m_os/blob/main/examples/README.md)

  <html>
  <small>
  <table>
  <tr><th>Session</th><th>Tema</th><th>:notebook: Slides</th><th>:clapper: Videos</th></tr>
  <tr><td rowspan="3">2</td>
      <td>Threads</td>
      <td><ul>
        <li> <a href="https://acaldero.github.io/uc3m_so/transparencias/clase_w6-hilos.pdf">hilos.pdf</a></li>
      </ul></td>
      <td>
      <ul type="1">
        <li><a href="https://youtu.be/n5qrEotEWfI">Introduction to threads</a></li>
        <li><a href="https://youtu.be/akf9UG7Z5Go">Main thread services</a></li>
      </ul>
      </td>
  </tr>
  <tr><td>Concurrency review</td>
      <td><ul>
        <li> <a href="https://acaldero.github.io/uc3m_so/transparencias/clase_w9-concurrencia-introduccion.pdf">concurrencia-introduccion.pdf</a> </li>
        <li> <a href="https://acaldero.github.io/uc3m_so/transparencias/clase_w10-concurrencia-servicios.pdf">concurrencia-servicios.pdf</a> </li>
      </ul></td>
      <td>
      <ul type="1">
        <li><a href="https://youtu.be/PxjgVYgpGkk">Introduction</a></li>
        <li><a href="https://youtu.be/EupaagvNpR0">POSIX synchronization mechanisms</a></li>
        <li><a href="https://youtu.be/8fdum4cvlvI">Simple producer-consumer example with POSIX mechanisms</a></li>
      </ul>
      </td>
  </tr>
  <tr><td>Concurrent servers</td>
      <td><ul>
        <li> <a href="https://acaldero.github.io/uc3m_so/transparencias/clase_w11-concurrencia-servidores.pdf">concurrencia-servidores.pdf</a> </li>
      </ul></td>
      <td>
      <ul type="1">
        <li><a href="https://youtu.be/nDyYrpFYG-4">Concurrent servers</a></li>
      </ul>
      </td>
  </tr>
  </table>
  </small>
  </html>
