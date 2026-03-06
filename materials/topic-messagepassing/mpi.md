# Communication with MPI
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

* Introduction to MPI:
    * [Defining MPI](#introduction-to-mpi)
    * [Main features](#main-features)
    * [Example in C: Hello world](#example-in-c-hello-world)
 * Point-to-point communication and collective communication:
    * [MPI API](#point-to-point-communication)
    * [Example in C: send and receive](#example-in-c-send-and-receive)
    * [Collective communication](#collective-communication)
    * [Example in C: broadcast and barrier](#example-in-c-broadcast-and-barrier)
  * Scalability:
    * [Example in C: calculating π with MPI](#example-in-c-calculating-π-with-mpi)
    * [Example in C: calculation of π with OpenMP](#example-in-c-calculating-%CF%80-with-mpi)


### Introduction to MPI

* "MPI is a message passing interface that represents a promising effort to improve the availability of highly efficient and portable software to meet current needs in high-performance computing through the definition of a universal message passing standard."
  * William D. Gropp et al.

* MPI stands for Message Passing Interface:
  * It is a standard message passing interface for developing parallel applications that run on computers that do not share memory (distributed memory).

* It is not an implementation but rather the specification that MPI implementations must comply with.
  The best known are:
  * OpenMPI 4.1.6 (9/30/2023) – http://www.open-mpi.org/
  * MPICH 4.1.2 (6/8/2023) – http://www.mpich.org/


### Key features

- **Portability**:
    - Defined independently of parallel platforms
    - Useful in heterogeneous parallel architectures
- **Efficiency**:
    - Defined for multithreaded applications
    - Based on reliable and efficient communication
    - Seeks to maximize each platform
- **Functionality**:
    - Easy to use for any programmer who has already used any message passing library


### MPI model

- Processes execute the same program
- All data structures and variables are local to each process
    - Processes execute in different address spaces (different machines)
    - Processes do not share memory
    - Processes exchange data by passing messages


### Example in C: Hello world

* **mpi_hello.c**
  ```c
  #include <stdio.h>
  #include “mpi.h”

  int main ( int argc, char **argv )
  {
       int comm_size, my_rank;
       int  tam = 255;
       char name[255];
       
       /* Initialize MPI, the first thing in the program (can modify argc and argv) */
       MPI_Init(&argc, &argv);
       
       MPI_Comm_size(MPI_COMM_WORLD, &comm_size); /* size <- how many processes there are */
       MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);   /* node <- my identifier, from 0 to (size-1) */
       
       MPI_Get_processor_name(name, &tam);
       printf("Process %d of %d processes (%s)\n", my_rank, comm_size, name);
       
       /* Finalize MPI, cannot be initialized again later */
       MPI_Finalize();
       return 0;
  }
  ```


* It can be compiled using `mpicc` as follows:
  ```bash
  mpicc -g -Wall -c mpi_hola.c -o mpi_hola.o
  mpicc -g -Wall -o mpi_hola mpi_hola.o
  ```

* It can be executed on the local machine using `mpirun`:
  1. First, create a `machines` file with the list of machines (one per line) that will be used for execution:
     ```bash
     cat <<EOF > machines
     localhost
     localhost
     EOF
     ```
  2. Then use `mpirun`:
     ```bash
     mpirun -np 2 -machinefile machines ./mpi_hola
     ```

* The output would be similar to:
  ```bash
  Process 0 of 2 processes (kiwi)
  Process 1 of 2 processes (kiwi)
  ```


### Point-to-point communication

The MPI API includes:
 - Data structures:
   * Data types (basic, vectors, composite, etc.)
   * Process groups (groups, communicators, etc.)
 - Message passing:
   * Point-to-point calls (blocking, asynchronous)
   * Collective calls (bcast, scatter, gather, etc.)
 - Input and output:
   * File management (opening, closing, etc.)
   * Content management (views, pointers, etc.)
 - Processes:
   * Process management (creation, etc.)
   * Profiling


### Example in C: send and receive

* **mpi_p2p.c**
  ```c
  #include <stdio.h>
  #include “mpi.h”

  int main ( int argc, char **argv )
  {
     int comm_size, my_rank;
     int num = 0;
    
     /* Initialize MPI, the first thing in the program (can modify argc and argv) */
     MPI_Init(&argc, &argv);
    
     MPI_Comm_size(MPI_COMM_WORLD, &comm_size); /* size <- how many processes there are */
     MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);   /* node <- my identifier, from 0 to (size-1) */

     if (my_rank == 0)
     {
        /*
           Send (MPI_Send) from a memory address (&num)
           one (1) integer (MPI_INT) to the process with rank 1 (1)
           in a message with label 0 by the MPI_COMM_WORD communicator 
        */
        num = 10 ;
        MPI_Send(&num, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);
     }
     else
     {
        /* 
           Receive (MPI_Recv) at a memory address (&num)
           an (1) integer (MPI_INT) from the process with rank 0 (0)
           in a message with label 0 by the communicator MPI_COMM_WORD
        */
        MPI_Recv(&num, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
        printf("[0 -> %d] num: %d\n", my_rank, num) ;
     }
    
     /* Finalize MPI, cannot be initialized again afterwards */
     MPI_Finalize();
     return 0;
  }
  ```

* It can be compiled using `mpicc`:
  ```bash
  mpicc -Wall -g mpi_p2p.c -o mpi_p2p
  ```

* It can be run on the local machine using `mpirun`:
  1. First, create a `machines` file with the list of machines (one per line) that will be used for execution:
     ```bash
     cat <<EOF > machines
     localhost
     localhost
     EOF
     ```
  2. Then use `mpirun`:
     ```bash
     mpirun -np 2 ./mpi_p2p
     ```

* The output would be similar to:
  ```bash
  [0 -> 1] num: 10
  ```


### Collective communication

Main primitives of collective communication:
* MPI_Barrier y MPI_Bcast:
  <html>
    <table border="0">
        <tr>
            <td><img alt="MPI_Barrier"   src="https://raw.githubusercontent.com/acaldero/uc3m_ds/main/materials/topic-messagepassing/mpi/mpi_barrier.svg" height="125"></td>
            <td><img width="25" height="1"></td>
            <td><img alt="MPI_Bcast"     src="https://raw.githubusercontent.com/acaldero/uc3m_ds/main/materials/topic-messagepassing/mpi/mpi_bcast.svg" height="150"></td>
        </tr>
    </table>
  </html>
* MPI_Scatter vs MPI_Gather:
  <html>
    <table border="0">
        <tr>
            <td><img alt="MPI_Scatter"   src="https://raw.githubusercontent.com/acaldero/uc3m_ds/main/materials/topic-messagepassing/mpi/mpi_scatter.svg" height="150"></td>
            <td><img width="25" height="1"></td>
            <td><img alt="MPI_Gather"    src="https://raw.githubusercontent.com/acaldero/uc3m_ds/main/materials/topic-messagepassing/mpi/mpi_gather.svg" height="150"></td>
        </tr>
    </table>
  </html>
* MPI_Gather vs MPI_Allgather:
  <html>
    <table border="0">
        <tr>
            <td><img alt="MPI_Gather"    src="https://raw.githubusercontent.com/acaldero/uc3m_ds/main/materials/topic-messagepassing/mpi/mpi_gather.svg" height="175"></td>
            <td><img width="25" height="1"></td>
            <td><img alt="MPI_Allgather" src="https://raw.githubusercontent.com/acaldero/uc3m_ds/main/materials/topic-messagepassing/mpi/mpi_allgather.svg" height="175"></td>
        <tr>
    </table>
  </html>
* MPI_Reduce vs MPI_Allreduce:
  <html>
    <table border="0">
        </tr>
            <td><img alt="MPI_Reduce"    src="http://hustcat.github.io/assets/mpi/mpi_reduce_00.png" height="150"></td>
            <td><img width="25" height="1"></td>
            <td><img alt="MPI_Allreduce" src="http://hustcat.github.io/assets/mpi/mpi_allreduce_00.png" height="150"></td>
        </tr>
    </table>
  </html>


### Example in C: broadcast and barrier

* **mpi_coll.c**
  ```c
  #include <stdio.h>
  #include "mpi.h"

  int main ( int argc, char **argv )
  {
      int comm_size, my_rank;
      int num = 0;
    
      /* Initialize MPI, first thing in the program (may modify argc and argv) */
      MPI_Init(&argc, &argv);
    
      MPI_Comm_size(MPI_COMM_WORLD, &comm_size); /* size <- how many processes there are */
      MPI_Comm_rank(MPI_COMM_WORLD, &my_rank);   /* node <- my identifier, from 0 to (size-1) */
    
      if (0 == my_rank)
           num = 5;  /* only process 0 has num with the value 5 */
      else num = 0 ; /* the rest of the processes have num with the value 0 */
    
      MPI_Bcast(&num, 1, MPI_INT, 0, MPI_COMM_WORLD);
      MPI_Barrier(MPI_COMM_WORLD);
      printf("Process %d has num: %d\n", my_rank, num);
    
      /* Finalize MPI, cannot be initialized again later */
      MPI_Finalize();
      return 0;
  }
  ```

* It can be compiled using `mpicc`:
  ```bash
  mpicc -Wall -g mpi_coll.c -o mpi_coll
  ```

* It can be run on the local machine using `mpirun`:
  1. First, create a `machines` file with the list of machines (one per line) that will be used for execution:
     ```bash
     cat <<EOF > machines
     localhost
     localhost
     EOF
     ```
  2. Then use `mpirun`:
     ```bash
     mpirun -np 2 ./mpi_coll
     ```

* The output would be similar to:
  ```bash
  Process 0 has num: 5
  Process 1 has num: 5
  ```


### Example in C: calculating π with MPI

* **pi_mpi.c**
  ```c
  #include <math.h>
  #include <stdio.h>
  #include <mpi.h>

  #define N 1E9
  #define d 1E-9

  int main(int argc, char* argv[])
  {
    int rank, size;
    double pi = 0.0, result = 0.0, sum = 0.0, s = 0, x = 0;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    for (int i = rank; i < N; i += size) {
        x = d * i;
        s = sqrt(4 * (1 - x * x)) * d;
        result = result + s;
    }

    MPI_Reduce(&result, &sum, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);

    if (rank == 0) {
        pi = 2 * sum;
        printf("np = %2d; PI = %lf\n", size, pi);
    }

    MPI_Finalize();
    return 0;
  }
  ```

* It can be compiled using `mpicc`:
  ```bash
  mpicc -Wall -g pi_mpi.c -lm -o pi_mpi
  ```

* It can be run on the local machine using `mpirun`:
  * First, create a `machines` file with the list of machines (one per line) that will be used for execution:
    ```bash
    cat <<EOF > machines
    localhost
    localhost
    EOF
    ```
  * Then use `mpirun`:
    ```bash
    mpirun -np 2 -machinefile machines ./pi_mpi
    ```

* The output would be similar to:
  ```bash
  np =  2; PI = 3.141593
  ```


### Example in C: calculating π with OpenMP

* **pi_omp.c**
  ```c
  #include <stdlib.h>
  #include <math.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <sys/time.h>
  #include <omp.h>

  #define N 1000000000
  #define d 1E-9

  int main ( int argc, char* argv[] )
  {
    long long i;
    double PI = 0.0, result = 0.0;

    #pragma omp parallel for reduction(+:result)
    for (i = 0; i < N; i++) {
        double x = d * i;
        result += sqrt(4 * (1 - x * x)) * d;
    }

    PI = 2 * result;
    printf("PI = %f\n", PI);

    return 0;
  }
  ```

* It can be compiled using:
  ```bash
  gcc -o pi_omp -fopenmp pi_omp.c -lm
  ```
* The OpenMP library `libopenmp.a` and the math library `libm.a` are required.

* It can be executed using:
  ```bash
  user$ ./pi_omp
  ```

* The output would be similar to:
  ```bash
  PI = 3.141593
  ```


### Further reading

* https://mpitutorial.com/tutorials/mpi-scatter-gather-and-allgather/
* http://hustcat.github.io/collective-communication-in-mpi/
* https://nyu-cds.github.io/python-mpi/05-collectives/

