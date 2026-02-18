
## Distributed Systems: Supplementary Materials
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Contents

 * [Support materials](#support-materials)
 * [Practical use cases](#practical-use-cases)
 * [Exercises](#exercises)


## Support materials

 <html>
 <small>
 <table>
  <tr><th>Topic</th><th>Lesson</th><th>Study materials</th></tr>
  <tr>
      <td rowspan="2">1</td>
      <td>C and make for distributed systems</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-candmake/ssdd_c.md">C aspects for distributed systems</a></li>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-candmake/ssdd_make.md">Introduction to make</a></li>
      </td>
  </tr>
  <tr>
      <td>Concurrenty on distributed systems</td>
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
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/ssdd_mpi.md">Introduction to MPI</a> 
             <a href="https://colab.research.google.com/github/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/ssdd_mpi.ipynb">(notebook)</a> </li>
        </li>
      </td>
  </tr>
  <tr>
      <td>
        Exercises:
        <ul>
        <li>
          <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/ejercicio_pasomensajes_vector.md">Vectors with POSIX queues</a>
          <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/e2-pasomensajes.pdf">(on PDF)</a>
        </li>
        <li> Fecha y hora <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/e1-pasomensajes.pdf">(on PDF)</a> </li>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-messagepassing/ejercicio_pasomensajes_upgraded.md">Naming and communication</a> </li>
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
       Ejercicio:
       <ul>
          <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-sockets/ejercicio_sockets_calculadora.md">Distributed calculator</a></li>
       </ul>
      </td>
  </tr>
  <tr>
      <td rowspan="1">5</td>
      <td>Distributed services</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-ds/ssdd_sd.md">Main distributed systems</a></li>
      </td>
  </tr>
  <tr>
      <td rowspan="1">6</td>
      <td>RPC</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-rpc/ssdd_rpc.md">RPC</a></li>
      </td>
  </tr>
  <tr><td>7</td>
      <td>Distributed file systems</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-dfs/ssdd_sfd.md">Introduction to distributed file systems</a></li>
      </td>
  </tr>
  <tr><td>8</td>
      <td>Web services</td>
      <td>
        <li> <a href="https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-ws/ssdd_web-services.md">Introduction to web services</a></li>
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

  * [Example of transforming a monolithic application into a distributed application: key-value storage](/materials/puc-keyvalue/#readme)
    * [Example of centralized monolithic key-value storage](/materials/puc-keyvalue/kv-centralizado-monolitico#readme)
    * [Example of distributed key-value storage based on POSIX queues](/materials/puc-keyvalue/kv-distribuido-mqueue#readme)
    * [Example of distributed key-value storage based on sockets](/materials/puc-keyvalue/kv-distribuido-sockets#readme)
    * [Example of distributed key-value storage based on RPC](/materials/puc-keyvalue/kv-distribuido-rpc#readme)

  * [Example of transforming a monolithic application into a distributed application: calculator](/materials/puc-calculator/#readme)
    * [Example of a centralized monolithic calculator](/materials/puc-calculator/cal-centralizado-monolitico#readme)
    * [Example of a distributed calculator based on POSIX queues](/materials/puc-calculator/cal-distribuido-mqueue#readme)
    * [Example of a distributed calculator based on sockets](/materials/puc-calculator/cal-distribuido-sockets#readme)
    * [Example of a distributed calculator based on RPC](/materials/puc-calculator/cal-distribuido-rpc#readme)
    * [Example of a distributed calculator based on GSOAP](/materials/puc-calculator/cal-distribuido-gsoap-standalone#readme)

