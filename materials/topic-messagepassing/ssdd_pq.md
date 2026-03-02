

# Communication with POSIX message queues
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

* Introduction to POSIX message queues
  * [Main features](#posix-message-queues)
* API for managing POSIX message queues:
  * [Create, open, close, and delete](#posix-calls-for-message-queue-management-13)     
     * [Example in C: create, close, and delete](#example-in-c-create-close-and-delete)
     * [Example in C: cleaning up queues that have NOT been deleted](#example-in-c-cleaning-up-queues-that-have-not-been-deleted)
  * [Send and receive](#posix-calls-for-message-queue-management-23)
     * [Example in C: producer-consumer with message queues](#example-in-c-producer-consumer-with-message-queues)
  * [Query and change attributes](#posix-calls-for-message-queue-management-33)
     * [Example in C: print the attributes of a POSIX queue](#example-in-c-print-the-attributes-of-a-posix-queue)
* [Sequential client-server with POSIX queues](#sequential-client-server-with-posix-queues)
* Other models
  * [Other message queue models](#other-message-queue-models)


### POSIX message queues

- **Implementation** of message passing in POSIX
- Communication **and** synchronization mechanism
- API similar to that used for files:
  * `mqd = mq_open("/storage.txt", O_CREAT | O_WRONLY, 0777, &attributes) ;`
  * `ret = mq_send    (mqd, (const char *)&data, sizeof (int), 0) ;`
  * `ret = mq_receive (mqd, (      char *)&data, sizeof(int), 0) ;`
  * `mq_close(mqd);`
  * `mq_unlink("/storage.txt");`
- Named:
  - **File names are assigned to queues** (`"/storage.txt"` in the previous examples of API usage)
    - **Only processes that share the same file system can use message queues**
    - The queue name allows it to be shared so that multiple processes can send and receive data
  - Indirect naming
- Synchronization:
  - Asynchronous sending
  - Synchronous or asynchronous reception
  - Blocking semantics (default) or non-blocking (`O_NONBLOCK` must be used)
- Communication:
  - **Unidirectional** data flow (two message queues are required for bidirectional)
  - Messages can be tagged with priorities
  - *Variable* message size


### POSIX calls for message queue management (1/3)

- **Create** and **open** a message queue
   ```c
   mqd_t mq_open ( char *name, int flag, mode_t mode, struct mq_attr *attr ) ;
   ```
  - Input arguments:
    - First argument: queue name
      - Follows file naming convention
      - ``man 7 mq_overview``
    - Second argument: queue **flags** (fnctl.h)
      - `O_CREAT`: creates a queue if it does not exist
      - `O_RDONLY`: creates a queue for receiving
      - `O_WRONLY`: creates a queue for sending
      - `O_RDWR`: creates a queue for receiving and sending
      - `O_NONBLOCK`: non-blocking sending and receiving
    - Third argument: queue access permissions (sys/stat.h)
      - `S_IRWXU`: read, write, and execute for owner
      - `S_IRWXU | S_IRWXG`: read, write, and execute for owner and group to which they belong 
      - `S_IRWXU | S_IRWXG | S_IRWXO`: read, write, and execute for owner, group to which it belongs, and other people on the machine
    - Fourth argument: queue attributes (```struct mq_attr attr```)
      - `attr.mq_maxmsg`: maximum number of queued messages
      - `attr.mq_msgsize`: maximum message size in bytes that can be sent to the queue
  - Returns:
    - An integer representing the queue descriptor or ```(mqd_t) -1``` if there is an error
- **Close** a message queue
  ```c
  int mq_close ( mqd_t mqdes ) ;
  ```
- **Delete** a message queue
  ```c
  int mq_unlink ( char *name ) ;
  ```
  * Remember: **Delete after closing** the message queue


### Example in C: create, close, and delete

* Example edition of creating a message queue
  
  **mq_create.c**
  ```c
  #include <stdio.h>
  #include <mqueue.h>
  #include <stdlib.h>
   
  // Tip: the name is an absolute path (starts with ‘/’)
  #define QUEUE_NAME "/storage.txt"

  int main ( int argc, char *argv[] )
  {
      mqd_t mqd;                   /* Queue descriptor */
      struct mq_attr attributes;   /* Queue attributes */
           
      attributes.mq_maxmsg = 10;           /* On Linux, this is usually 10 for unprivileged processes */
      attributes.mq_msgsize = sizeof(int); /* Size of each message in bytes */
       
      mqd = mq_open(QUEUE_NAME, O_CREAT | O_WRONLY, 0777, &attributes) ;
      if (-1 == mqd) {
          perror(“mq_open: ”);
          exit(-1);
      }
       
      // ...Use the POSIX message queue...

      mq_close(mqd);
      mq_unlink(QUEUE_NAME);
  }
  ```

* To compile, you can use:
  ```bash
  gcc -Wall -g mq_create.c -lrt -o mq_create
  ```
  - You need to link with the **`librt`** library

* To run, you can use:
  ```bash
  user$ ./mq_create
  ```


### Example in C: cleaning up queues that have NOT been deleted

* It is possible that a program working with POSIX queues may fail and leave a queue undeleted.
  This can be a problem, so let's look at how to solve it with an example (`mq_nounlink.c`).
    
  **mq_nounlink.c**
  ```c
  #include <stdio.h>
  #include <mqueue.h>
  #include <stdlib.h>
   
  #define QUEUE_NAME "/storage.txt"

  int main ( int argc, char *argv[] )
  {
       mqd_t mqd;                   /* Queue descriptor */
       struct mq_attr attributes;   /* Queue attributes */
           
       attributes.mq_maxmsg = 10;           
       attributes.mq_msgsize = sizeof(int); 
       mqd = mq_open(QUEUE_NAME, O_CREAT | O_WRONLY, 0777, &attributes) ;
       
       if (-1 == mqd) {
           perror(“mq_open: ”);
           exit(-1);
       }
       // <- Do not delete with unlink(QUEUE_NAME)!
   }
  ```

* To compile, you can use:
  ```bash
  gcc -Wall -g mq_nounlink.c -lrt -o mq_nounlink
  ```
  - You need to link with the **`librt`** library

* To run, you can use:
  ```bash
  user$ ./mq_nounlink   
  ```

* **Important**: POSIX queues can be visible from the command line:
  ``` bash
  user$ sudo mkdir /dev/mqueue
  user$ sudo mount -t mqueue none /dev/mqueue
  user$ ls -las /dev/mqueue
   
  total 0
  0 drwxrwxrwt  2 root   root     60 Jul  4 11:35 .
  0 drwxr-xr-x 12 root   root   3180 Jul  4 11:39 ..
  0 -rwx------  1 user   user     80 Jul  4 11:33 store.txt

  user$ cat /dev/mqueue/store.txt
  QSIZE:0          NOTIFY:0     SIGN:0     NOTIFY_PID:0
  ```

* **Important**: There is a maximum number of POSIX queues that can be created at one time (256 in Linux/Ubuntu 24.04):
  ``` bash
  user$ cat /proc/sys/fs/mqueue/queues_max
  256
  ```
  
* **Important**: If unused POSIX queues are not deleted, there may come a time when no more can be created. To avoid this, it is possible to delete unused POSIX queues from the command line:
  ``` bash
  user$ rm /dev/mqueue/*
  user$ ls -las /dev/mqueue/
  total 0
    
  0 drwxrwxrwt  2 root   root     60 Jul  4 11:35 .
  0 drwxr-xr-x 12 root   root   3180 Jul  4 11:39 ..
  ```


### POSIX calls for message queue management (2/3)

- **Send** a message to a queue
  ```c
  int mq_send ( mqd_t mqdes, const char *msg, size_t len, unsigned int prio ) ;
  ```
  - The message `msg` of length `len` bytes is sent to the message queue `mqdes` with priority `prio`
  - Input arguments:
    -   `mqdes`: Queue descriptor
    -   `msg`: Buffer containing the message to be sent
    -   `len`: Length of the message to be sent
    -   `prio`: Message priority Messages are inserted according to their priority (`0 ... MQ_PRIOMAX`)   
  - Returns: 
    - 0 if successful or -1 if an error occurs
    - If the queue is full AND the send is non-blocking (O_NONBLOCK), then 
      the process does not block but returns -1 (EAGAIN)

- **Receive** a message from a queue
  ```c
  int mq_receive ( mqd_t mqdes, char *msg, size_t len, unsigned int *prio ) ;
  ```
  - Receives the message `msg` with the highest priority in the queue (`prio`) of length `len` bytes from the queue `mqdes`
  - Input arguments:
    -   `mqdes`: Queue descriptor
    -   `msg`: Buffer containing the message to be received
    -   `len`: Length of the message to be received
    -   `prio`: Priority of the message received
  - Returns:
    - The number of bytes in the message received or -1 in case of error.
    - If the queue is empty AND reception is non-blocking (O_NONBLOCK), then 
      the process does not block but returns -1 (EAGAIN)


### Example in C: producer-consumer with message queues

General design idea:
 
* At the process level
  ```mermaid
   flowchart LR
       Producer --> Consumer
  ```
* At the code level within each process
   <html>
   <table border="1">
   <tr>
   <td>
   <pre>
   Producer() 
   {
       for(;;) {
            &lt;Produce data&gt;
            <b>send</b>(Consumer, data);
       } /* end for */
   }
   </pre>
   </td>
   <td>
   <pre>
   Consumer()
   {
       for(;;) {
            <b>receive</b>(Producer, data);
            &lt;Consume data&gt;
        } /* end for */
   }
   </pre>
   </td>
   </tr>
   </table>
   </html>


**producer.c**
```c
#include <mqueue.h>
#include <stdio.h>
#include <stdlib.h>

#define MAX_BUFFER 10
#define DATA_TO_PRODUCE 100000

void Producer ( mqd_t queue )
{
    int data;
    int i, ret;
        
    for (i = 0; i < DATA_TO_PRODUCE; i++)
    {
        data = i; /* Produce data */
        ret = mq_send(queue, (const char *)&data, sizeof(int), 0) ;
        
        if (ret == -1) {
            perror(“mq_send: ”);
            mq_close(queue);
            exit(1);
        }
    }
}

int main ( int argc, char *argv[] )
{
    /* storage: message queue where produced data is left and data to be consumed is extracted */
    mqd_t storage;
    struct mq_attr attr;
    
    attr.mq_maxmsg = MAX_BUFFER;
    attr.mq_msgsize = sizeof(int);

    storage = mq_open("/STORAGE", O_CREAT | O_WRONLY, 0700, &attr) ;
    if (storage == -1) {
        perror("mq_open: ");
        exit(-1);
    }

    Producer(storage);

    mq_close(storage);
    return (0);
}
```

**consumer.c**
```c
#include <mqueue.h>
#include <stdio.h>
#include <stdlib.h>

#define MAX_BUFFER 10
#define DATA_TO_PRODUCE 100000

void Consumer ( mqd_t queue )
{
    int data;
    int i, ret;
    for (i = 0; i < DATA_TO_PRODUCE; i++)
    {
        ret = mq_receive(queue, (char *)&data, sizeof(int), 0) ;
        
        if (ret == -1) {
            perror("mq_receive: ");
            mq_close(queue);
            exit(1);
        }
        
        printf("The data consumed is: %d\n", data);
    }
}

int main ( int argc, char *argv[] )
{
    mqd_t store;

    if ((store = mq_open("/STORE", O_RDONLY)) == -1) {
        perror("mq_open: ");
        exit(-1);
    }

    Consumer(store);
        
    mq_close(storage);
    return (0);
}
```

* To compile, you can use:
  ```bash
  gcc -Wall -g producer.c -lrt -o producer
  gcc -Wall -g consumer.c -lrt -o consumer
  ```

* To run, you can use:
  ```bash
  user$ ./producer &
  user$ ./consumer

  The data consumed is: 0
  The data consumed is: 1
  ...
  The data consumed is: 99997
  The data consumed is: 99998
  The data consumed is: 99999
  [1]+  Done                    ./producer
  ```
   

### POSIX calls for message queue management (3/3)

- **Modify** the attributes of a queue
  ```c
  int mq_setattr(mqd_t mqdes, struct mq_attr *qstat, struct mq_attr *oldmqstat) ;
  ```
  - Input arguments:
     -   `mqdes`: Queue descriptor
     -   `qstat`: New queue attributes
     -   `oldmqqstat`: If oldmqstat is not NULL, the old attributes will be stored in it
  - Returns:
    - -1 if there is an error

- **Get** the attributes of a queue
  ```c
  int mq_getattr (mqd_t mqdes, struct mq_attr *qstat) ;
  ```
  - Input arguments:
    - `mqdes`: Queue descriptor
    - `qstat`: Queue attributes
  - Returns:
    - -1 if there is an error
    - `qstat` modified with the queue attributes:
      - `mq_maxmsg`: Maximum number of messages
      - `mq_msgsize`: Maximum message size
      - `mq_curmsgs`: Current number of messages in the queue
      - `mq_flags`: Flags associated with the queue


### Example in C: print the attributes of a POSIX queue

**mq_attr.c**
```c
#include <stdio.h>
#include <mqueue.h>
#include <stdlib.h>

int main ( int argc, char *argv[] )
{
    mqd_t mqd;
    struct mq_attr attributes;

    mqd = mq_open("/storage.txt", O_CREAT | O_WRONLY, 0777, &attributes) ;
    if (mqd == -1) {
        printf(“mq_open error\n”);
        exit(-1);
    }
    
    if (mq_getattr(mqd, &attributes) == -1) {
        printf("Error mq_getattr\n");
        exit(-1);
    }

    printf("Maximum number of messages in the queue: %ld\n",     attributes.mq_maxmsg) ;
    printf("Maximum message length in the queue: %ld\n",         attributes.mq_msgsize) ;
    printf("Number of messages currently in the queue: %ld\n",   attributes.mq_curmsgs) ;
    
    mq_close(mqd);
    exit(0);
}
```

* To compile, you can use:
  ```bash
  gcc -Wall -g mq_attr.c -lrt -o mq_attr
  ```

* To run, you can use:
  ```bash
  user$ ./mq_attr

  Maximum number of messages in the queue: 10
  Maximum length of the message in the queue: 4
  Number of messages currently in the queue: 0
  ```


### Sequential client-server with POSIX queues

General design idea:
* At the process level:
  ```mermaid
  flowchart LR
  Client <--struct request--> Server
  ```
* At the code level within each process:
  <html>
  <table border="1">
  <tr>
  <td>
  <pre>
  Client() 
  {
      request.client_queue <- unique name
      request.operation  <- value
      request.arguments <- values
      ...
      sq = mq_open(SERVER_QUEUE, ...)
      cq = mq_open(request.client_queue, ...)
      <b>send</b>(sq, &request);
      <b>receive</b>(cq, &response, ...)
      mq_close(sq)
      mq_close(cq)
      mq_unlink(request.client_queue)
      ...
      work with response...
  }
  </pre>
  </td>
  <td>
  <pre>
  Server()
  {
     sq = mq_open(SERVER_QUEUE, ...)
     while(1) {
          <b>receive</b>(sq, &request, ...)        
          ...
          response <- process request
          ...
          cq = mq_open(request.client_queue, ...)
          <b>send</b>(cq, &response, ...)
          mq_close(cq)
     } /* end for */
  }
  </pre>
  </td>
  </tr>
  </table>
  </html>


**message.h**
```c
#define MAXSIZE 256
#define SUM   0
#define SUBTRACT 1
// . . .

struct request {
   int op; /* operation, 0 (+) 1 (-) */
   int  a; /* operand 1 */
   int  b; /* operand 2 */
   char q_name[MAXSIZE]; /* name of the client queue
                            where the server should send the response */
};
```


**server.c**
```c
#include "message.h"
#include <mqueue.h>

int main ( int argc, char *argv[] )
{
   mqd_t q_server;  /* server message queue */
   mqd_t q_client;  /* client message queue */
   struct mq_attr attr;
   struct request pet;
   int res;
   
   attr.mq_maxmsg = 10;
   attr.mq_msgsize = sizeof(struct request);
   server_queue = mq_open("/SERVER_SUM", O_CREAT|O_RDONLY, 0700, &attr);
   while(1)
   {
       mq_receive(q_server, (char *) &pet, sizeof(pet), 0);
       if (pet.op ==0) 
            res = pet.a + pet.b;
       else res = pet.a - pet.b;

       /* respond to the client by first opening its queue */
       client_queue = mq_open(pet.q_name, O_WRONLY);
       mq_send(client_queue, (const char *)&res, sizeof(int), 0);
       mq_close(client_queue);
   }
}
```


**client.c**
```c
#include <string.h>
#include <stdio.h>
#include <mqueue.h>
#include <unistd.h>
#include "message.h"

int main ( int argc, char *argv[] )
{
   mqd_t q_server;  /* message queue for the server process */
   mqd_t q_client;  /* message queue for the client process */
   struct mq_attr attr;
   struct request pet;
   int res; 
   char queuename[MAXSIZE];

   attr.mq_maxmsg = 1;
   attr.mq_msgsize = sizeof(int);
   sprintf(queuename, "/CLIENT-%d", getpid()) ; /* generate a unique name for each client */
   q_client = mq_open(queuename, O_CREAT|O_RDONLY, 0700, &attr);
   q_server = mq_open("/SERVER_SUM", O_WRONLY);

   /* fill in the request and send it */
   pet.op = 0;
   pet.a = 5;
   pet.b = 2;
   strcpy(pet.q_name, queuename);
   mq_send(q_server, (const char *)&pet, sizeof(pet), 0);

   /* receive the response and print it */
   mq_receive(q_client, (char *) &res, sizeof(int), 0);
   printf("Result = %d\n", res);

   /* Close and delete the POSIX queues */
   mq_close(q_server);
   mq_close(q_client);
   mq_unlink(queuename);

   return 0;
}
```


* To compile, you can use:
  ```bash
  gcc -Wall -g server.c -lrt -o server
  gcc -Wall -g client.c -lrt -o client
  ```

* To run, you can use:
  ```bash
  user$ ./server &
  user$ ./client
  Result = 7
          
  user$  kill -9 %1
  ```



### Other message queue models

-   Advanced Message Queueing Protocol (AMQP)
-   Apache ActiveMQ
-   IBM MQ
-   Java Message Service
-   Microsoft Message Queuing

