
## Distributed Systems: Supplementary Materials
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

 * [Support materials](#support-materials)
 * [Practical use cases](#practical-use-cases)


## Support materials

 <html>
 <small>
 <table>
  <tr><th>Topic</th><th>Lesson</th><th>Study materials</th></tr>
  <tr>
      <td rowspan="1">0</td>
      <td>C and make for distributed systems</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-candmake/ssdd_c.md">C aspects for distributed systems</a></li>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-candmake/ssdd_make.md">Introduction to make</a></li>
      </td>
  </tr>
  <tr>
      <td rowspan="1">1</td>
      <td>Introduction</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-intro/intro.md">Introduction to distributed systems</a></li>
      </td>
  </tr>
  <tr>
      <td rowspan="1">2</td>
      <td>Communication and synchronization</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-concurrency/ssdd_threads_posix.md">Concurrency based on POSIX</a></li>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-concurrency/ssdd_threads_c.md">Concurrency base on C<sub>11</sub></a></li>
      </td>
  </tr>
  <tr>
      <td rowspan="2">3</td>
      <td rowspan="2">Message passing</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/ssdd_pq.md">POSIX message queues</a> </li>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/mpi.md">Introduction to MPI</a> 
             <a href="https://colab.research.google.com/github/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/mpi.ipynb">(notebook)</a> </li>
      </td>
  </tr>
  <tr>
      <td>
        Exercises:
        <ul>
        <li>
          <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/exercise_messagepassing_vector.md">Vectors with POSIX queues</a>
          <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/e2-pasomensajes.pdf">(in PDF)</a>
        </li>
        <li> Date and hour <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/e1-pasomensajes.pdf">(in PDF)</a> </li>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/exercise_messagepassing_upgraded.md">Naming and communication</a> </li>
        </ul>
      </td>
  </tr>
  <tr>
      <td rowspan="2">4</td>
      <td rowspan="2">Sockets</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-sockets/ssdd_sockets.md">Sockets</a></li>
      </td>
  </tr>
  <tr>
      <td>
       Exercise:
       <ul>
          <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-sockets/exercise_sockets_calculator.md">Distributed calculator</a></li>
       </ul>
      </td>
  </tr>
  <tr>
      <td rowspan="1">5</td>
      <td>Distributed services</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-ds/ds.md">Main distributed systems</a></li>
      </td>
  </tr>
  <tr>
      <td rowspan="1">6</td>
      <td>RPC</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-rpc/rpc.md">RPC</a></li>
      </td>
  </tr>
  <tr><td>7</td>
      <td>Distributed file systems</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-dfs/dfs.md">Introduction to distributed file systems</a></li>
      </td>
  </tr>
  <tr><td>8</td>
      <td>Web services</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-ws/web-services.md">Introduction to web services</a></li>
      </td>
  </tr>
  <tr><td>9</td>
      <td>Fault tolerance</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-ft/t9_tolerancia-a-fallos.pdf">t9_tolerancia-a-fallos.pdf</a> :notebook_with_decorative_cover:</li>
      </td>
  </tr>
 </table>
 </small>
</html>


## Practical use cases

### Transforming a monolithic application into a distributed application:

<html>
<ul><small>
<table width="100%">
<tr><th>Storage <br>key-value</th>
<td>
</html>

 [Key steps for transforming](/materials/pc-keyvalue/#readme) a storage application (key-value) from <br> a monolithic architecture to a distributed architecture:
 ```mermaid
  %%{init: {"flowchart": {"diagramPadding": 80}}}%%
  flowchart LR
    A[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-keyvalue/kv-centralized-monolithic#readme'>monolithic</a>]
    B[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-keyvalue/kv-centralized-library#readme'>monolithic with<br> library</a>]
    C{proxy <br>pattern <br>with...}
    D[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-keyvalue/kv-distributed-mqueue#readme'>POSIX queues</a>]
    E[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-keyvalue/kv-distributed-sockets#readme'>sockets</a>]
    F[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-keyvalue/kv-distributed-rpc#readme'>RPC</a>]
    A --> B
    B --> C
    C -- mqueue  --> D
    C -- sockets --> E
    C -- RPC     --> F
  ```

<html>
</td></tr>
<tr><th>Calculator</th>
<td>
</html>

 [Key steps for transforming](/materials/pc-calculator/#readme) a computational application (calculator) from <br> a monolithic architecture to a distributed architecture:
 ```mermaid
  %%{init: {"flowchart": {"diagramPadding": 80}}}%%
  flowchart LR
    A[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-calculator/cal-centralized-monolitico#readme'>monolithic</a>]
    B[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-calculator/cal-centralized-library#readme'>monolithic with<br> library</a>]
    C{proxy <br>pattern <br>with...}
    D[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-calculator/cal-distributed-mqueue#readme'>POSIX queues</a>]
    E[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-calculator/cal-distributed-sockets#readme'>sockets</a>]
    F[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-calculator/cal-distributed-rpc#readme'>RPC</a>]
    G[<a href='https://github.com/acaldero/uc3m_ds/tree/main/materials/pc-calculator/cal-distributed-gsoap-standalone#readme'>gSOAP</a>]
    A --> B
    B --> C
    C -- mqueue  --> D
    C -- sockets --> E
    C -- RPC     --> F
    C -- gSOAP   --> G
  ```

<html>
</td></tr>
</table>
</small></ul>
</html>

