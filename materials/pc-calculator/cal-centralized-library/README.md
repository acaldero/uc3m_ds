
## Distributed Systems: Supplementary Materials
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Centralized service with library

### To compile 

Please execute this first:
```
cd cal-centralized-library
make
```

And the expected output should be:
```
gcc -g -Wall -c app-c.c
gcc -g -Wall -c lib.c
gcc -g -Wall app-c.o lib.o  -o app-c
```

### To execute

Please execute this:
```
./app-c
```

And the expected output should be similar to:
```
0 = add(30, 20, 10)
0 = divide(2, 20, 10)
0 = neg(-10, 10)
```

### Architecture

```mermaid
sequenceDiagram
    app-c   ->> lib.c: request lib.h API
    lib.c   ->> app-c: return result of API call
```

