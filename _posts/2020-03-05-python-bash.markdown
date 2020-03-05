# Run shell with python

https://stackabuse.com/executing-shell-commands-with-python/

## subprocess

Call run method in subprocess module, with *CompletedProcess* returned.

```python
>>> import subprocess
>>> cmds = "hdfs dfs -test -d /user/hive"
>>> subprocess.run(cmds.split())
CompletedProcess(args=['hdfs', 'dfs', '-test', '-d', '/user/hive'], returncode=0)
>>> cmds = "hdfs dfs -test -d /user/hive1"
>>> subprocess.run(cmds.split())
CompletedProcess(args=['hdfs', 'dfs', '-test', '-d', '/user/hive1'], returncode=1)
>>> subprocess.run(cmds.split(),  stdout=subprocess.PIPE)
CompletedProcess(args=['hdfs', 'dfs', '-test', '-d', '/user/hive1'], returncode=1, stdout=b'')

#if shell=True, the first parameter should text instead of list
completed_process = subprocess.run(cmd_clear_hdfs_data, stdout=subprocess.PIPE, shell=True)

```



## params

### shell

If shell=True, the first parameter should be *text* instead of  list. It's not recommended for not secure. But if you need "*" in bash command, you need to take this way.



For shell=False,  the parameter should be *list*, for example ['hdfs', 'dfs', '-ls' , '/user/hive']



### stdout

if you set stdout=subprocess.PIPE, you can get it from CompleteProcess object.
