

n-postgresql

https://www.postgresql.org/

## key words

Postgresql, python

## 简介

postgresql是另一种关系型数据库



## postgresql的常用命令

不同于mysql， postgresql没有show databases, show tables, exit等命令，而使用另一些种类的命令。

连接：使用psql客户端, https://www.postgresqltutorial.com/psql-commands/

显示数据库：\d

显示表：\dt

```shell
$psql -d dvdrental -U postgres -W
Password for user postgres:
#List available databases
\l
#Switch connection to a new database
\c dbname username
#List available tables
\dt
#Describe a table
\d table_name
```



## 读取
与mysql类似，读取需要模块中的connect函数，需要连接的cursor(),需要execute

get connection

get cursor

execute(sql)

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
import psycopg2
import json

#读取sql中的内容，并写到文件内
#注意读取出的dict写到文件中可能包含单引号的问题
def extract_data_by_sql(file_path, sql):
    host, user, password, dbname = get_config()

    with psycopg2.connect(host=host, user=user, password=password, dbname=dbname) as con:
        csr = con.cursor()
        csr.execute(sql)
        rows = csr.fetchmany(10000)
        batch=0
        with open(file_path, 'w') as f:
            while rows:
                if batch>0:#注意，这里需要换行
                    f.write('\n')    
                data = '\n'.join(['\t'.join([json.dumps(col) if isinstance(col, dict) or isinstance(col, list) else str(col) for col in row]) for row in rows])#对于读取出的数据中，存在dict和list的情况，需要特别注意，因为他们的str表示包含的是单引号
                f.write(data)
                batch += 1
                rows = csr.fetchmany(10000)
```






