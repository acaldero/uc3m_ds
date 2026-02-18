
## Distributed Systems: Supplementary Materials
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Monolithic centralized service

### To compile 

Please execute this first:
```
cd cal-centralizado-monolitico
make
```

And the expected output should be:
```
gcc -g -Wall -c app-c.c
gcc -g -Wall app-c.o  -o app-c
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
    app-c ->> app-c: request API call and return result of API call
```

