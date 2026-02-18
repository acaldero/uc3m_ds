
## Distributed Systems: Supplementary Materials
+ **Felix García Carballeira and Alejandro Calderón Mateos** @ arcos.inf.uc3m.es
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_ds/blob/main/LICENSE)


## Fault tolerant support based on Python Checkpointing

#### Preparation

Please execute this first:
```
cd lab-ft-checkpoint
./run.sh
```


#### To execute

<html>
<table>
<tr><th>Step</th><th>Client</th></tr>

<tr>
<td>1</td>
<td>

```
./run.sh
+ python3 app.py
iter:  0
iter:  1
iter:  2
iter:  3
iter:  4
🧨 🧨
```

</td>
</tr>

<tr>
<td>2</td>
<td>

```
./run.sh
+ python3 app.py
iter:  4
iter:  5
iter:  6
iter:  7
iter:  8
iter:  9
🧨 🧨
```

</td>
</tr>

</table>
</html>


