
## Distributed Systems: Supplementary Materials
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Distributed service based on POSIX queues

#### To compile

Please, first execute:
```
cd cal-distributed-mqueue
make
```

And the output should be similar to:
```
gcc -g -Wall -c app-d.c
gcc -g -Wall -c lib-client.c
gcc -g -Wall -c lib.c
gcc -g -Wall -lrt app-d.o lib.o lib-client.o       -o app-d  -lrt
gcc -g -Wall -c lib-server.c
gcc -g -Wall    lib.o lib-client.o lib-server.o  -o lib-server  -lrt
```

#### To execute

*TIP: POSIX queues are used to communicate processes on the same machine*

<html>
<table>
<tr><th>Step</th><th>Client</th><th>Server</th></tr>
<tr>
<td>1</td>
<td></td>
<td>

```
$ ./lib-server
```

</td>
</tr>

<tr>
<td>2</td>
<td>

```
$ ./app-d
0 = d_add(30, 20, 10)
0 = d_divide(2, 20, 10)
0 = d_neg(-10, 10)
```

</td>
<td>

```
 0 = d_add(30, 20, 10)
 0 = d_divide(2, 20, 10)
 0 = d_neg(-10, 10)
```

</td>
</tr>

<tr>
<td>3</td>
<td></td>
<td>

```
^Caccept: Interrupted system call
```

</td>
</tr>
</table>
</html>

*TIP: POSIX queues can be viewed from the command line:*

``` bash
sudo mkdir /dev/mqueue
sudo mount -t mqueue none /dev/mqueue
ls -las /dev/mqueue
```

#### Architecture

```mermaid
sequenceDiagram
    app-d          ->> lib-client.c: request lib.h API in a distributed way
    lib-client.c   ->> lib-server.c: request remote API
    lib-server.c   ->> lib.c: request lib.h API call
    lib.c          ->> lib-server.c: return API call result
    lib-server.c   ->> lib-client.c: return remote result
    lib-client.c   ->> app-d: return result of the distributed API call
```

