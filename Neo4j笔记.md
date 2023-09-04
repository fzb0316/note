---
typora-root-url: ./
---

# Neo4j





### 1 介绍和语法

Cypher是一种类似与SQL的查询语言

#### 1.1 值和类型

**属性类型：**

Number(Integer、float)、String、Boolean、Point

Temporal types:Data、Time、LocalTime、DataTime、*LocalDateTime* 、*Duration*

以及上述类型的List

**结构类型：**

Nodes：Id、Label(s)、Map

Relationships：Id、Type、Map、Id of the start and end nodes

Paths：Nodes和Relationships的交替序列

**复合类型：**

null：空

List：异构、有序，可以是属性、结构、或复合类型

Map：key-alue、无序，key是String，value可以是属性、结构、或复合类型

#### 1.2 表达式

**CASE表达式**

```cypher
MATCH (n)                 
RETURN
CASE n.eyes
WHEN 'blue'
THEN 1
WHEN 'brown'
THEN 2
ELSE 3 END AS result
```

```cypher
MATCH (n)
RETURN
CASE
WHEN n.eyes = 'blue'
THEN 1
WHEN n.age < 40
THEN 2
ELSE 3 END AS result
```

#### 1.3 参数

支持带参查询 `$param`，可以配合正则表达式(=~)使用

参数也可以表示Nodes

```javascript
{
  "props" : [ {
    "awesome" : true,
    "name" : "Andy",
    "position" : "Developer"
  }, {
    "children" : 3,
    "name" : "Michael",
    "position" : "Developer"
  } ]
}
```

```cypher
UNWIND $props AS properties
CREATE (n:Person)
SET n = properties
RETURN n
```

若是单个Node，可以直接使用

```cypher
CREATE ($props)
```

#### 1.4 运算符

**DISTINCT**：消除查询结果中的重复值

**.** ：点运算符静态访问节点或关系的属性

**[]**：动态访问节点或关系的属性：`restaurant["rating_" + category.name]`

​		`name[1..3]`返回name的List中的1到3号索引，左闭右开

**=**：直接赋值，删掉之前所有的属性

**+=**：是增加属性，也会更改本次赋值中之前已经存在的但值不同的属性

**数字运算符**：+ - * / ^ %

**比较运算符**：=、>、<、<>、<=、>=、IS NULL、IS NOT NULL

(比较可以任意链接、`x < y <= z`相当于 `x < y AND y <= z`.)

**字符串搜索**：   STARTS WITH(前缀)、ENDS WITH(后缀)、CONTAINS(包含)

**布尔运算**：AND、OR、XOR、NOT

**+**：连接字符串，连接两个List

**=~**：匹配正则表达式

**IN**：类似与python的IN

#### 1.5 注释：`//`

#### 1.6 模式与模式匹配

**()表示Node（a,b,c,r为变量可以省略）**

```cypher
(a)、(a:User)、(a:User {name:"李四",...})、(a:User:Person)
```

**[]表示关系**

```cypher
(a)-[r:friend {from:2018,...}]->(b)
```

**-->表示方向**

```cypher
(a)-->(b)、(a)-->(b)<--(c)、(a)--(b)、(a)-[r]->(b)
(a)-[r:TYPE1|TYPE2]->(b)//关系只有一种标签，这种表示为type1或2
(a)-[*2]->(b)//两跳邻居（有向），也可无向
(a)-[*3..5]->(b)//指定长度范围
(a)-[*3..]->(b)  (a)-[*..5]->(b)
(a)-[*]->(b)
```

#### 1.7 时间和日期（暂时没看）



#### 1.8 空间（point——坐标），未看，后期补充



#### 1.9 List

**表示：使用方括号**

```cypher
RETURN [0, 1, 2, 3, 4, 5, 6, 7, 8, 9] AS list
```

**支持range——range(1,10)表示一个列表，闭区间**

```cypher
RETURN range(0, 10)[3]
RETURN range(0, 10)[-3]
RETURN range(0, 10)[0..3]//类python、[]中左闭右开
```

**Size函数：**

```cypher
RETURN size(range(0, 10)[0..3])//结果为3
```

**表达式构建：**

```cypher
RETURN [x IN range(0,10) WHERE x % 2 = 0] AS result
```

### 2 常用命令

#### 2.1 MATCH

##### 2.1.1 基础节点查找

- 获取所有节点

```cypher
MATCH (n) RETURN n
```

- 获取所有带标签的节点

```cypher
MATCH (movie:Movie) RETURN movie
MATCH (n:student),(m:teacher) RETURN n,m
```

- 获取相关节点(邻居)(使用Movie这个标签作了限制，也可以不写标签)

```cypher
MATCH (n:Person { name: 'Keanu Reeves' })--(movie:Movie)
RETURN n,movie
```

##### 2.1.2 基础关系查询

- 出边指向的节点

```cypher
MATCH (:Person  { name: 'Keanu Reeves' })-->(movie)
RETURN movie
```

- 查询边

```cypher
MATCH (:Person  { name: 'Keanu Reeves' })-[r]->(:Movie)
RETURN type(r),r
```

- 通过关系的类型(一种或者多种，也可以使用变量r)来查询

```cypher
MATCH (n:Movie { title: 'RescueDawn' })<-[r:ACTED_IN|DIRECTED]-(person)
RETURN person,r.roles
```

- 可变长的关系查询(多跳邻居)

```cypher
match (n:student {name:"小明"})-[:isfriend*2]-(m:student)
return m//2跳邻居，当然，这里的关系是可以有向的
                                
match (n:student {name:"小明"})-[:isfriend*1..3]-(m:student)
return m//1到3跳邻居
                                
match p=(n:student {name:"小明"})-[:isfriend*2]-(m:student)
return p//返回所有关系列表  
                                               
MATCH p =(m:student)-[[:isfriend *]-(n:student)
WHERE m.name = '小明' AND n.name = '小胖'
RETURN p                     
```

- 命名路径

```cypher
MATCH p =(michael { name: 'Michael Douglas' })-->()
RETURN p
```

- 通过id查询（也适用于Node的查询）

```cypher
MATCH (a)-[r]-(b)
WHERE id(r)= 0
RETURN a,b
```

##### 2.1.3 最短路径

- 单一最短路径

```cypher
MATCH (m:Person { name: 'J.T. Walsh' }),(n:Person { name: 'Regina King' }), p = shortestPath((m)-[*..15]-(n))
RETURN p
```

- 带谓词的单一最短路径

```cypher
MATCH (m:Person { name: 'J.T. Walsh' }),(n:Person { name: 'Regina King' }), p = shortestPath((m)-[*]-(n))
WHERE NONE (r IN relationships(p) WHERE type(r)= 'REVIEWED')
RETURN p
```

- 所有最短路径

```cypher
MATCH (m:Person { name: 'J.T. Walsh' }),(n:Person { name: 'Regina King' }), p = allShortestPaths((m)-[*]-(n))
RETURN p
```

- 单源最短路径

```cypher
MATCH (m:Person { name: 'J.T. Walsh' }),(n), p = shortestPath((m)-[*]-(n))
where not id(n) = id(m)
RETURN p
```



##### 2.1.4 可选匹配`OPTIONAL MATCH`

与MATCH不同的是，如果查询不到，就返回一个NULL

```cypher
MATCH (a:Movie { title: 'Jerry Maguire' })
OPTIONAL MATCH (a)-->(x)
RETURN x,x.name  //可选关系，没有出边的话返回NULL
                      
MATCH (a:Movie { title: 'Jerry Maguire' })
OPTIONAL MATCH (a)-[r:ACTS_IN]->()
RETURN a.title, r//可选关系的命名
```

##### 2.1.5 高级查询(我自己总结的)

- **查询出度数量：**

```cypher
match (n:Person) with n,size((n)-[]->()) as s return n.name, s
```

（具体查询某个人物的话根普通查询一样，查询入度的话类似）

- **查询度：**

```cypher
match (n:student{name:"小明"})-[r]-() return count(r)
```

- **查询多跳邻居的数量：**

```cypher
match (n:Person {name:"Keanu Reeves"})-->(m:Movie) with n,m,size((n)-->(m)<--()) as s return n,m,s

match (n:Person {name:"Keanu Reeves"}) with n,size((n)-[]->()<-[]-()) as s return n,s
```

#### 2.2 RETURN

- 返回Node

```cypher
MATCH (n { name: '小明' })
RETURN n
```

- 返回关系

```cypher
MATCH (n { name:  '小明' })-[r:isfriend]->(c)
RETURN r
```

- 返回属性

```cypher
MATCH (n :student{age:12})
RETURN n.name
```

- 返回所有元素（节点、关系、路径）

```cypher
MATCH p =(a { name: '小明' })-[r]->(b)
RETURN *
```

- 引入特殊字符需要使用占位符：\

```cypher
MATCH (`This isn\'t a common variable`)
WHERE `This isn\'t a common variable`.name = 'A'
RETURN `This isn\'t a common variable`.happy
```

- 列别名：使用`AS`

```cypher
MATCH (a { name: '小明' })
RETURN a.feat AS feature
```

- 其他返回

```cypher
MATCH (a { name: '张三' })
RETURN a.age > 30, "I'm a literal",(a)-->()
```

- 使用`DISTINCT`消除重复

```cypher
MATCH (a { name: '小明' })-[*2]-(b)
RETURN DISTINCT b
```

#### 2.3 WITH

- 过滤聚合的结果

```cypher
MATCH (david { name: '小明' })--(otherPerson)-->()
WITH otherPerson, count(*) AS foaf
WHERE foaf > 1
RETURN otherPerson.name//小明的邻居且大于一个出度的点
```

- 使用collect之前对结果排序

```cypher
MATCH (n:student)
WITH n
ORDER BY n.name DESC LIMIT 3
RETURN collect(n.name)//collect是返回一个列表
```

- 路径搜索限制

```cypher
MATCH (n { name: '小明' })--(m)
WITH m
ORDER BY m.name DESC LIMIT 1
MATCH (m)--(o)
RETURN distinct o
```

#### 2.4 UNWIND

展开列表的操作，返回每一个值

- 基本操作

```cypher
UNWIND [1, 2, 3, NULL ] AS x
RETURN x  //返回四个值

WITH [[1, 2],[3, 4], 5] AS nested
UNWIND nested AS x
UNWIND x AS y
RETURN y      //展开双重列表
```

- 与`DISTINCT`、`collect`搭配去重

```cypher
WITH [1, 1, 2, 2] AS coll
UNWIND coll AS x
WITH DISTINCT x
RETURN collect(x) AS setOfVals          //返回[1,2]
```

#### 2.5 WHERE

- 布尔运算

```cypher
MATCH (n:student)
WHERE n.name = '小明' XOR (n.age < 30 AND n.name = '张三') OR NOT (n.name = '张三' OR n.name = '小明')
RETURN n.name, n.age
```

- 过滤节点标签、节点属性、关系属性

```cypher
MATCH (n)
WHERE n:student
RETURN n.name, n.age

MATCH (n)
WHERE n.age < 30
RETURN n.name, n.age

MATCH (n)-[k:isfriend]->(f)
WHERE k.since > 2013
RETURN f.name, f.age
```

- 过滤动态计算的节点属性

```cypher
WITH 'AGE' AS age
MATCH (n:student)
WHERE n[toLower(age)]< 14   //toLower是变为小写
RETURN n.name, n.age  //返回年龄小于14的student
```

- 属性存在性检查

```cypher
MATCH (n:student)
WHERE exists(n.feat)
RETURN n.name, n.feat //只会返回有feat属性的student
```

- 字符串匹配

```cypher
MATCH (n:Person)
WHERE n.name STARTS WITH 'James'
RETURN n.name, n.born  //名字以James开头的
       
MATCH (n:Person)
WHERE n.name ENDS WITH 'son'
RETURN n.name, n.born
       
MATCH (n:Person)
WHERE /*NOT*/n.name CONTAINS 'ch' //加NOT的话就是否定
RETURN n.name, n.born   //子串匹配
       
MATCH (n)
WHERE n.name =~ 'La.*' //正则表达式，也支持转义字符，加上(?i)代表不区分大小写
RETURN n.name, n.age
```

- 过滤路径模式

```cypher
MATCH (m:student { name: '小明' }),(n)
WHERE n.name IN ['小红', '张三'] AND (m)-->(n)
RETURN n.name, n.age
```

#### 2.6 order by

默认升序，使用DESC降序，NULL属于最小的

#### 2.7 LIMIT与SKIP

LIMIT：输出前n行 `LIMIT n`

SKIP：跳过前n行 `SKIP n`

可以配合使用，达到输出中间部分的效果

#### 2.8 CREATE

- 创建节点

```cypher
//创建简单节点
create (n)
//创建多个节点
create (n),(m)
//创建带标签和属性的节点并返回节点
create (n:person {name:'如来'}) return n
```

- 创建关系

  单向关系：-->；双向关系:--；创建只能创建单向关系

```cypher
//使用新节点创建关系
CREATE (n:person {name:'杨戬'})-[r:师傅]->(m:person {name:'玉鼎真人'}) return typr(r)
//使用已知节点创建带属性的关系
match (n:person {name:'沙僧'}),(m:person{name:'唐僧'})
create (n)-[r:'师傅'{relation:'师傅'}]->(m) return r
```

#### 2.9 DELETE

删除之前需要查询，也可以只删除关系

```cypher
//删除学生小飞
MATCH (n:Person { name: '小飞' })
DELETE n
//删除所有节点
MATCH (n)
DETACH DELETE n    
//删除带关系的节点
MATCH (n { name: 'Andy' })
DETACH DELETE n
//只删除关系
MATCH (n { name: '小明' })<-[r:isfriends]-()
DELETE r
```

#### 2.10 SET

- 向现有节点或关系添加新属性
- 添加或更新属性值

```cypher
MATCH (n:student {name:"小红"}) set n.feat=[0,0,0,0,0,1,0,0,0]
return n
                   
MATCH (n { name: '李四' })
SET (
CASE
WHEN n.age = 36
THEN n END ).worksIn = 'Malmo'
RETURN n.name, n.worksIn
//更改类型
MATCH (n { name: '李四' })
SET n.age = toString(n.age),n.name = "王五"//也可以同时set多个
RETURN n.name, n.age
//删除属性
MATCH (n { name: '李四' })//或者SET p = {},删除所有属性
SET n.name = NULL RETURN n.name, n.age
//替换所有属性（使用Map）
MATCH (p { name: '李四' })
SET p = { name: '王五', age=10 }//值得注意的是，=号时，若不给age赋值，那么age则被置为NULL，与之不同的是+=
RETURN p.name, p.age
//设置标签
MATCH (n { name: '李四' })
SET n:German
RETURN n.name, labels(n) AS labels
```

#### 2.11 REMOVE

删除属性，标签(节点的)

```cypher
MATCH (a { name: '李四' })
REMOVE a.age//删除age
RETURN a.name, a.age
//删除标签
MATCH (n { name: '李四' })
REMOVE n:German
RETURN n.name, labels(n)
```

#### 2.12 FOREACH

```cypher
//标记路径上的节点
MATCH p =(begin)-[*]->(END )
WHERE begin.name = 'A' AND END .name = 'D'
FOREACH (n IN nodes(p)| SET n.marked = TRUE )
```

#### 2.13 MERGE

MATCH+CREATE

#### 2.14 CALL

call相当于启动一个子查询

```cypher
CALL {
  MATCH (p:Person)
  RETURN p
  ORDER BY p.age ASC
  LIMIT 1
UNION
  MATCH (p:Person)
  RETURN p
  ORDER BY p.age DESC
  LIMIT 1
}
RETURN p.name, p.age
ORDER BY p.name
```

#### 2.15 UNION

与SQL一样

#### 2.16 LOAD CSV

```cypher
//将csv拷贝到 %NEO4J_HOME%\import目录
load csv from 'file:///西游记.csv' as line
create (:西游 {name:line[0],tail:line[1],label:line[3]})
```

### 3 Functions

见官网，主要提供的都是基本函数，

### 4 执行计划

#### 4.1 查询树

执行的查询任务被分解为不同的*operators*，每一个*operator*执行一项特定的，所有的*operators*组成一个树，称为执行计划。每个*operator*都表示为树中的一个节点，一个运算符的输出是其父节点的输入，最后由树根产生查询结果；每一个*operator*都是一个查询计划算子，直接从存储引擎获取数据——数据库命中

大多数*operators*在产生结果后就将其输出至管道（传递给父节点），但是一些运算符（比如排序）需要聚合所有的结果，这种算子会导致高内存，影响性能

#### 4.2 查询计划算子详解

https://neo4j.com/docs/cypher-manual/4.4/execution-plans/operators/

第十二条以后

总结就是：系统会统计一些数据，来调整合适的查询计划。总的来说就是每一个*operator*尽可能的过滤掉多的行数。

#### 4.3 最短路径算法执行计划

```cypher
PROFILE
MATCH (KevinB:Person {name: 'J.T. Walsh'} ),
      (Al:Person {name: 'Stephen Rea'}),
      p = shortestPath((KevinB)-[*]-(Al))
RETURN p
```

这种查询两个节点之间的最短路径，官网说的是neo4j使用双向广度优先搜索的算法

查询执行计划：`PROFILE`        `EXPLAIN`(不执行，只看计划)

执行计划是通过优化的，但是也可以通过手动调优，而shortestPath()有可能会触发遍历所有路径的可能（当存在谓词约束时）

```cypher
PROFILE
MATCH (KevinB:Person {name: 'J.T. Walsh'} ),
      (Al:Person {name: 'Stephen Rea'}),
      p = shortestPath((KevinB)-[*]-(Al))
where length(p) > 1
RETURN p
```

防止穷举：

```cypher
PROFILE
MATCH  (KevinB:Person {name: 'J.T. Walsh'} ),
      (Al:Person {name: 'Hugo Weaving'}),
      p = shortestPath((KevinB)-[*]-(Al))
WITH p
WHERE length(p) > 1
RETURN p
```

### 5 查询调优

手动调优主要是率先过滤多数的信息

#### 5.1 简单查询调优

```cypher
PROFILE
MATCH (p {name: 'Tom Hanks'})
RETURN p
```

运行发现，执行计划首先过滤了ALLNODES，添加一个标签就会快很多，随着数据的增多，这种快会越来越明显

```cypher
PROFILE
MATCH (p:Person {name: 'Tom Hanks'})
RETURN p
```

但是随着时间的增加，查询会再次变慢

```cypher
CREATE INDEX FOR (p:Person)
ON (p.name)
    
CALL db.awaitIndexes
//添加索引后，执行计划就减少到一行了
PROFILE
MATCH (p:Person {name: 'Tom Hanks'})
RETURN p
```

#### 5.2 高级查询调优

```cypher
PROFILE
MATCH (p:Person)-[:ACTED_IN]->(m:Movie)
WHERE p.name STARTS WITH 'Tom'
RETURN
  p.name AS name,
  count(m) AS count
```

