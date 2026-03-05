
# Example of a distributed calculator based on TCP sockets
+ **Felix Garcia Carballeira and Alejandro Calderon Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

* [Statement](#statement)
* [Guide to help solve distributed application design exercises](#guide-to-help-solve-distributed-application-design-exercises)
* [Example of socket-based distributed calculator design](#example-of-a-distributed-calculator-design-based-on-sockets)
* [Example of distributed calculator service implementation with TCP](#example-of-distributed-calculator-service-implementation-with-tcp)


## Statement

Design and implement a reliable distributed calculator service that allows addition and subtraction in C11 using sockets.


## Guide to help solve distributed application design exercises

<html>
<table>
<tr>
<td>
<ol type="1" start="1">
<li>Identify the client and server parts
<ul>
<li> Client: active element, there may be several
<li> Server: passive element
</ul>
<li>Identify the service protocol
<ul>
<li>Identify requests/responses
<li>Identify message types
<li>Identify message exchange sequence
</ul>
</ol>
</td>
<td>
What
</td>
</tr>

<tr>
<td>
<ol type="1" start="3">
<li>Choose the type of server:
<ul>
<li> Connectionless (UDP)
<li> Connection-oriented (TCP)
<ul>
<li> Connection by request
<li> Connection by session
</ul>
</ul>
</ol>
</td>
<td>
How (1/2)
</td>
</tr>

<tr>
<td>
<ol type="1" start="4">
<li>Define the message format (data representation) <br>
and the detailed message passing sequence
<ul>
<li>Seek independence wherever possible (language, architecture, implementation, etc.)
</ul>
</ol>
</td>
<td>
Initial design
</td>
</tr>

<tr>
<td>
<ol type="1" start="5">
<li>Design concurrency aspects (sequential, heavy process, light process on demand, or light process with pool)
<li>Naming (static or dynamic addressing)
</ol>
</td>
<td>
How (2/2)
</td>
</tr>

<tr>
<td>
<ol type="1" start="7">
<li> Modify message format and modify detailed message passing sequence if necessary
</ol>
</td>
<td>
Final design
</td>
</tr>
</table>
</html>


## Example of a distributed calculator design based on sockets

1. Identify the client and server parts:
   * **Client: active element that, when executed on machine A, sends a request over the network.**
   * **Server: passive element that, when executed on machine B, is in a loop in which (1) it receives the request from a client over the network, (2) processes it, and (3) responds with the result.**

2. Identify the service protocol (requests/responses, types, and exchange sequence):
   ```mermaid
   sequenceDiagram
   Client1 ->>+ Server: addition/subtraction request
   Note right of Server: processes request
   Server ->>- Client1: addition/subtraction result

   Client2 ->>+ Server: addition/subtraction request
   Note left of Server: processes request
   Server ->>- Client2: addition/subtraction result
   ```

3. Server type: connection-oriented with request-based connection is used:
   * **TCP: We don't want the exchanged messages to be lost**
   * **Per request: A session with multiple requests in that request is not required**

4. Definition of the message format (data representation) and the detailed message passing sequence:
   ```mermaid
   sequenceDiagram
   Client1 ->>+ Server: Operation code (1 byte)
   Client1 ->> Server: First operand (4 bytes, network format)
   Client1 ->> Server: Second operand (4 bytes, network format)
   Note right of Server: Process request
   Server ->>- Client1: Result (4 bytes, network format)

   Client2 ->>+ Server: Operation code (1 byte)
   Client2 ->> Server: First operand (4 bytes, network format)
   Client2 ->> Server: Second operand (4 bytes, network format)
   Note left of Server: Process request
   Server ->>- Client2: Result (4 bytes, network format)
   ```

5. Design concurrency aspects (sequential, heavy process, light process on demand, light process with pool)
   * **Sequential server.**

6. Naming (static or dynamic addressing)
   * **"Static" addressing: the server listens on port 4200 and the client must be given the IP address of the machine.**

8. Modify the message format and modify the detailed message passing sequence if necessary
   * **For the sequential server and static addressing, it is not necessary to modify the diagram in step 4.**



## Example of distributed calculator service implementation with TCP

* One possible implementation would be based on the following structure:

    ![Calculator implementation structure](/materials/topic-sockets/exercise_sockets_calculator/calc-arq.svg)

  * calc-server-tcp.c -> implementation of a calculator service with TCP sockets.
  * calc-client-tcp.c -> implementation of a calculator client with TCP sockets.
  * comm.h -> communications library interface.
  * comm.c -> implementation of the communications library.


* To compile, you can use:
  ```bash
  gcc -I./ -Wall -g -c comm.c
  gcc -I./ -Wall -g -c calc-server-tcp.c
  gcc -I./ -Wall -g -c calc-client-tcp.c
  gcc -o calc-client-tcp calc-client-tcp.o comm.o
  gcc -o calc-server-tcp calc-server-tcp.o comm.o
  ```

* The files would be:

<details open>
<summary><b>comm.h</b></summary>

```c
#ifndef _COMM_H_
#define _COMM_H_

  #include <sys/types.h>
  #include <sys/socket.h>
  #include <arpa/inet.h>
  #include <netdb.h>
  #include <unistd.h>
  #include <stdio.h>
  #include <string.h>
  #include <errno.h>

  int serverSocket ( unsigned int addr, int port, int type ) ;
  int serverAccept ( int sd ) ;
  int clientSocket ( char *remote, int port ) ;
  int closeSocket ( int sd ) ;

  int sendMessage ( int socket, char *buffer, int len );
  int recvMessage ( int socket, char *buffer, int len );

  ssize_t writeLine ( int fd, char *buffer ) ;
  ssize_t readLine ( int fd, char *buffer, size_t n );

#endif
```
</details>


<details open>
<summary><b>calc-server-tcp.c</b></summary>

```c
  #include <unistd.h>
  #include <stdio.h>
  #include <strings.h>
  #include <signal.h>
  #include "comm.h"

  int service ( int sc )
  {
     int ret ;
     char op;
     int32_t a, b, res;

     ret = recvMessage(sc, (char *) &op, sizeof(char)); // operation
     if (ret < 0) {
         printf("Error receiving op\n");
         return -1 ;
     }

     ret = recvMessage(sc, (char *) &a, sizeof(int32_t)); // receives a
     if (ret == -1) {
         printf("Error receiving a\n");
         return -1 ;
     }
     a = ntohl(a); // unmarshalling

     ret = recvMessage(sc, (char *) &b, sizeof(int32_t)); // receive b
     if (ret == -1) {
         printf("Error receiving b\n");
         return -1;
     }
     b = ntohl(b); // unmarshalling

     res = 0;
     if (0 == op) res = a + b;
     if (1 == op) res = a - b;
     if (2 == op) res = a * b;
     if (3 == op) res = a / b;

     res = htonl(res); // marshalling
     ret = sendMessage(sc, (char *)&res, sizeof(int32_t));
     if (ret == -1) {
         printf("Error sending\n");
         return -1 ;
     }

     return 0 ;
  }

int the_end = 0;

void sigHandler ( int sign )
{
     the_end = 1 ;
}

int main ( int argc, char *argv[] )
{
    int sd, sc;
    struct sigaction new_action, old_action;

    // create socket
    sd = serverSocket(INADDR_ANY, 4200, SOCK_STREAM);
    if (sd < 0) {
        printf("SERVER: Error in serverSocket\n");
        return 0;
    }

    // if Ctrl-C is pressed, the loop ends
    new_action.sa_handler = sigHandler;
    sigemptyset(&new_action.sa_mask);
    new_action.sa_flags = 0;
    sigaction(SIGINT, NULL, &old_action);
    if (old_action.sa_handler != SIG_IGN) {
        sigaction(SIGINT, &new_action, NULL);
    }

    while (0 == the_end)
    {
        // accept connection with client
        sc = serverAccept(sd);
        if (sc < 0) {
            printf("Error in serverAccept\n");
            continue;
        }

        // process request
        service(sc);

        // close connection with client
        closeSocket(sc);
    }

    closeSocket(sd);
    return 0;
}
```

</details>


<details open>
<summary><b>calc-client-tcp.c</b></summary>

```c
  #include <unistd.h>
  #include <stdio.h>
  #include <string.h>
  #include <stdlib.h>
  #include "comm.h"

  int remote_sum ( int sd, int *r, int x, int y )
  {
    int ret;
    char op;
    int32_t a, b, res ;

    op = 0; // add operation
    ret = sendMessage(sd, (char *) &op, sizeof(char)); // send operation
    if (ret == -1) {
        printf("Error sending op\n");
        return -1;
    }

    a = htonl(x); // marshalling: a <- host to network long(x)
    ret = sendMessage(sd, (char *) &a, sizeof(int32_t)); // send a
    if (ret == -1) {
        printf("Error sending a\n");
        return -1;
    }

    b = htonl(y); // marshalling: b <- host to network long(y)
    ret = sendMessage(sd, (char *) &b, sizeof(int32_t)); // send b
    if (ret == -1) {
        printf("Error sending b\n");
        return -1;
    }

    ret = recvMessage(sd, (char *) &res, sizeof(int32_t)); // receive response
    if (ret == -1) {
        printf("Error receiving\n");
        return -1;
    }
    *r = ntohl(res); // unmarshalling: *r <- network to host long(res)

    return 0 ;
  }

  int main ( int argc, char **argv )
  {
    int sd, ret, res;

    if (argc != 3) {
        printf("Usage: %s <server address> <server port>\n", argv[0]);
        printf("Example -> %s localhost 4200\n\n", argv[0]);
        return(0);
    }

    char *host = argv [1];
    int port = atoi(argv[2]);

    sd = clientSocket(host, port);
    if (sd < 0) {
        printf("Error in clientSocket with %s:%d\n", host, port);
        return -1;
    }

    ret = remote_sum(sd, &res, 5, 2);
    if (ret < 0) {
        closeSocket(sd);
        return -1;
    }

    printf("Result of a+b is: %d\n", res);

    closeSocket(sd);
    return 0;
  }
```

</details>


<details open>
<summary><b>comm.c</b></summary>

```c
#include "comm.h"

int serverSocket ( unsigned int addr, int port, int type )
{
   struct sockaddr_in server_addr ;
   int sd, ret;

   // Create socket
   sd = socket(AF_INET, type, 0);
   if (sd < 0) {
      perror("socket: ");
      return (0);
   }

   // Option to reuse address
   int val = 1;
   setsockopt(sd, SOL_SOCKET, SO_REUSEADDR, (char *) &val, sizeof(int));

   // Address
   bzero((char *)&server_addr, sizeof(server_addr));
   server_addr.sin_family = AF_INET;
   server_addr.sin_addr.s_addr = INADDR_ANY;
   server_addr.sin_port = htons(port);

   // Bind
   ret = bind(sd, (const struct sockaddr *)&server_addr, sizeof(server_addr));
   if (ret == -1) {
      perror("bind: ");
      return -1;
   }

   // Listen
   ret = listen(sd, SOMAXCONN);
   if (ret == -1) {
      perror("listen: ");
      return -1;
   }

   return sd ;
}

int serverAccept ( int sd )
{
   int sc ;
   struct sockaddr_in client_addr ;
   socklen_t size ;

   printf("waiting for connection...\n");

   size = sizeof(client_addr);
   sc = accept(sd, (struct sockaddr *)&client_addr, (socklen_t *)&size);
   if (sc < 0) {
      perror("accept: ");
      return -1;
   }

   printf("connection accepted from IP: %s and port: %d\n",
           inet_ntoa(client_addr.sin_addr), ntohs(client_addr.sin_port));

   return sc;
}

int clientSocket ( char *remote, int port )
{
   struct sockaddr_in server_addr;
   struct hostent *hp;
   int sd, ret;

   sd = socket(AF_INET, SOCK_STREAM, 0);
   if (sd < 0) {
      perror("socket: ");
      return -1;
   }

   hp = gethostbyname(remote);
   if (hp == NULL) {
      printf("Error in gethostbyname\n");
      return -1;
   }

   bzero((char *)&server_addr, sizeof(server_addr));
   memcpy (&(server_addr.sin_addr), hp->h_addr, hp->h_length);
   server_addr.sin_family = AF_INET;
   server_addr.sin_port = htons (port); 

   ret = connect(sd, (struct sockaddr *) &server_addr, sizeof(server_addr));
   if (ret < 0) {
      perror("connect: ");
      return -1;
   }

   return sd ;
}

int closeSocket ( int sd )
{
   int ret ;

   ret = close(sd) ;
   if (ret < 0) {
      perror("close: ");
      return -1;
   }

   return ret ;
}

int sendMessage ( int socket, char * buffer, int len )
{
   int r;
   int l = len;

   do
   {
      r = write(socket, buffer, l);
      if (r < 0) {
         return (-1); /* error */
      }
      l = l -r;
      buffer = buffer + r;

   } while ((l>0) && (r>=0));

   return 0;
}

int recvMessage ( int socket, char *buffer, int len )
{
   int r;
   int l = len;

   do {
      r = read(socket, buffer, l);
      if (r < 0) {
         return (-1); /* error */
      }
      l = l -r ;
      buffer = buffer + r;

   } while ((l>0) && (r>=0));

   return 0;
}

ssize_t writeLine ( int fd, char *buffer )
{
   return sendMessage(fd, buffer, strlen(buffer)+1) ;
}

ssize_t readLine ( int fd, char *buffer, size_t n )
{
   ssize_t numRead; /* bytes read in last read(...) */ 
 
   size_t totRead;  /* bytes read so far */
   char *buf;
   char ch;

   if (n <= 0 || buffer == NULL) {
      errno = EINVAL;
      return -1;
   }

   buf = buffer;
   totRead = 0;

   while (1)
   {
      numRead = read(fd, &ch, 1); /* read one byte */

      if (numRead == -1) {
          if (errno == EINTR) /* interruption -> restart read() */
               continue;
          else return -1; /* other type of error */
      } else if (numRead == 0) { /* EOF */
          if (totRead == 0) /* no bytes read -> return 0 */
               return 0;
          else break;
      } else { /* numRead must be 1 here */
          if (ch == ‘\n’) break;
          if (ch == ‘\0’) break;
          if (totRead < n - 1) { /* discard > (n-1) bytes */
              totRead++;
              *buf++ = ch;
          }
      }
   }

   *buf = ‘\0’;
   return totRead;
}
```

</details>



* To run on a machine, you can use:
  ```bash
  $ ./calc-server-tcp &
  waiting for connection...

  $ ./calc-client-tcp
  Usage: ./calc-client-tcp <server address> <server port>
  Example -> ./calc-client-tcp localhost 4200

  $ ./calc-client-tcp localhost 4200
  connection accepted from IP: 127.0.0.1 and port: 41356
  waiting for connection...
  Result of a+b is: 7

  $ ./calc-client-tcp localhost 4200
  connection accepted from IP: 127.0.0.1 and port: 41368
  waiting for connection...
  Result of a+b is: 7

  $ ./calc-client-tcp localhost 4201
  connect: : Connection refused
  Error in clientSocket with localhost:4201

  $ pkill -n -f calc-server-tcp
  ```

* To run on two machines, you can use:
  <table>
  <tr><td></td><td> host-a </td> <td> host-b </td></tr>
  <tr>
  <td>1. Start server</td>
  <td width=50%></td>
  <td width=50%>

  ```bash
  $ ./calc-server-tcp &
  waiting for connection...
  ```

  </td>
  </tr>
  <tr>
  <td>2.- Run client</td>
  <td><small>

  ```bash
  $ ./calc-client-tcp
  Usage: ./calc-client-tcp <address> <port>
  Example -> ./calc-client-tcp localhost 4200

  $ ./calc-client-tcp host-b 4200
  connection accepted from IP: 127.0.0.1 and port: 41356
  waiting for connection...
  Result of a+b is: 7
  ```

  </small></td>
  <td> 
  </td>
  </tr>
  <tr>
  <td>3.- Terminate server</td>
  <td></td>
  <td>

  ```bash
  $ pkill -n -f calc-server-tcp
  ```

  </td>
  </tr>
  </table>
