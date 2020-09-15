---
1title: MongoDB(一)
tags: [DB,MongoDB]
categories: [DB]
date: 2020-09-03 17:40:00
---

# MongoDB (一)

>MongoDB（来自于英文单词“Humongous”，中文含义为“庞大”）是可以应用于各种规模的企业、各个行业以及各类应用程序的开源数据库。作为一个适用于敏捷开发的数据库，MongoDB的数据模式可以随着应用程序的发展而灵活地更新。与此同时，它也为开发人员 提供了传统数据库的功能：`二级索引`，`完整的查询系统`以及`严格一致性`等等。 MongoDB能够使企业更加具有`敏捷性`和`可扩展性`，各种规模的企业都可以通过使用MongoDB来创建新的应用，提高与客户之间的工作效率，加快产品上市时间，以及降低企业成本。
>
>MongoDB是专为`可扩展性`，`高性能`和`高可用性`而设计的数据库。它可以从`单服务器`部署扩展到大型、复杂的`多数据中心`架构。利用`内存计算`的优势，MongoDB能够提供`高性能的数据读写操作`。 MongoDB的`本地复制`和`自动故障转移`功能使您的应用程序具有企业级的`可靠性`和`操作灵活性`。

## 安装

MongoDB有两个版本，社区版和企业版。（以下使用ubuntu社区版）

> version 4.4

### 社区版

> 不支持WSL

1. 包管理添加公钥

    ```shell
    ~ ❯ wget -qO - https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -   06:31:54 PM
    OK
    ```

    如果执行失败，提示gnupg未安装，则先执行安装gnupg命令，然后重试。

    ```shell
    sudo apt-get install gnupg
    ```

2. 生成列表文件

   * ubuntu 18.04

   ```shell
   echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu bionic/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
   ```

   * ubuntu 20.04

   ```shell
   echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
   ```

3. 更新

    ```shell
    sudo apt-get update
    ```

4. 下载

   * 最新版本

   ```shell
   sudo apt-get install -y mongodb-org
   ```

   * 指定版本

   ```shell
   sudo apt-get install -y mongodb-org=4.4.0 mongodb-org-server=4.4.0 mongodb-org-shell=4.4.0 mongodb-org-mongos=4.4.0 mongodb-org-tools=4.4.0
   ```

    >将MongoDB固定到当前版本，避免升级包时候误升级。
    >
    >```shell
    >echo "mongodb-org hold" | sudo dpkg --set-selections
    >echo "mongodb-org-server hold" | sudo dpkg --set-selections
    >echo "mongodb-org-shell hold" | sudo dpkg --set-selections
    >echo "mongodb-org-mongos hold" | sudo dpkg --set-selections
    >echo "mongodb-org-tools hold" | sudo dpkg --set-selections
    >```

其他版本安装参考：[官方文档](https://docs.mongodb.com/manual/installation/)

## MongoShell

> MongoShell是由JavaScript实现的MongoDB客户端命令行工具，可以用来增删改查，执行权限操作等。
>
> MongoDB 4.2 (and 4.0.13)之后连接非官方MongoDB，会有提示警告信息。
>
> 限制每行4095 个codepoints ，超过会被截断。

1. 本地连接

    直接用mongo命令连接，默认端口`27017`

    ```shell
    mongo
    ```

    指定端口

    ```shell
    mongo --port 28028
    ```

2. 远程连接

    * 需要明确指定主机名和端口号

    * 字符串

    ```shell
    mongo "mongodb://mongodb0.example.com:28015"
    ```

    * mongo --host <host>:<port>

    ```shell
    mongo --host mongodb0.example.com:28015
    ```

    * mongo --host <host> --port <port>

    ```
    mongo --host mongodb0.example.com --port 28015
    ```

3. 需要身份验证的实例

    * 字符串

    ```shell
    mongo "mongodb://alice@mongodb0.examples.com:28015/?authSource=admin"
    ```

    > 可以不指定密码，shell会提示输入密码。

    * --username <username> --password --authenticationDatabase <db>

    ```shell
    mongo --username alice --password --authenticationDatabase admin --host mongodb0.examples.com --port 28015
    ```

    > 指定--password参数，而没有输入密码，shell同样会提示输入密码。

4. 连接集群

    * 字符串(一组连接字符串','分割)

    ```shell
    mongo "mongodb://mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017/?replicaSet=replA"
    ```

    * 命令 --host <set name>/<host>:<port>,<host>:<port>

    ```shell
    mongo --host replA/mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017
    ```

5. TLS/SSL连接

    * 字符串指定`ssl=true`参数

    ```shell
    mongo "mongodb://mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017/?replicaSet=replA&ssl=true"
    ```

    * DNS种子列表连接

    ```shell
    mongo "mongodb+srv://server.example.com/"
    ```

    > 使用+ srv连接字符串修饰符会自动将ssl选项设置为true。

    * 命令行 指定 --ssl参数

    ```shell
    mongo --ssl --host replA/mongodb0.example.com.local:27017,mongodb1.example.com.local:27017,mongodb2.example.com.local:27017
    ```

    mongo shell 使用：

    * 显示当前使用的库

    ```shell
    db
    ```

    * 列出所有库

    ```shell
    show dbs
    ```

    * 切换库

    ```shell
    use <database>
    ```

    > [`db.getSiblingDB()`](https://docs.mongodb.com/manual/reference/method/db.getSiblingDB/#db.getSiblingDB)从当前数据库访问其他数据库而不切换当前db
    >
    > 可以直接切到不存在的库，新插入数据时候会自动新建库及集合；只切新库，不执行插入操作是不会新建库及集合的。
    >
    > ```shell
    > use myNewDatabase
    > db.myCollection.insertOne( { x: 1 } );
    > ```
    >
    > * db 当前数据库 
    > * myCollection 集合名
    > *   insertOne() 插入记录方法
    >
    > 如果集合名含有空格、-、数字开头或者是与内置函数冲突，则不能用db.<collection name>的方式选中集合，使用db.getCollection()代替。（db.getCollection('my-Collection')）

    * 列出当前库集合

    ```shell
    show collections
    ```

    输出结果格式化：

    ```shell
    db.myCollection.find().pretty()
    ```

    其他格式化方法：

    print() ,printjson() , print(tojson(<obj>))

    qiut(); ctrl c 退出mongo shell

    其他操作文档连接：

    - [Getting Started Guide](https://docs.mongodb.com/getting-started/shell)
    - [Insert Documents](https://docs.mongodb.com/manual/tutorial/insert-documents/)
    - [Query Documents](https://docs.mongodb.com/manual/tutorial/query-documents/)
    - [Update Documents](https://docs.mongodb.com/manual/tutorial/update-documents/)
    - [Delete Documents](https://docs.mongodb.com/manual/tutorial/remove-documents/)
    - [mongo Shell Methods](https://docs.mongodb.com/manual/reference/method/)

### 样式配置

1. 自定义提示

   对prompt进行设置，可以是字符串或者是JavaScript代码。可以在当前mongo shell中设置 也可以在`/etc/mongorc.js`文件中进行设置。

   * 显示行号
   ![image-20200915181642262](/img/mongodb/image-20200915181642262.png)
     
   * 显示db和hostname
   ![image-20200915181715722](/img/mongodb/image-20200915181715722.png)
     
   * 显示系统正常运行时间和当前文档数量
   ![image-20200915181738149](/img/mongodb/image-20200915181738149.png)
   
       
   
2. 使用外部编辑器
   
    启动mongo shell之前设置`EDITOR`环境变量

    ```shell
    export EDITOR=vim
    mongo
    ```

    在mongo shell中用 edit \<function> ,edit\<variable>来打开外部编辑器进行编辑。
    
3. 设置Batch Size （default number of items to display on shell）
   
    ```shell
    DBQuery.shellBatchSize = 10;
    ```


### 查看帮助

1. mongo 参数帮助

    ```shell
    mongo --shell
    ```

2. mongo shell中细分的其他帮助

    ```shell
    > help
            db.help()                    help on db methods # db相关的方法帮助文档
            db.mycoll.help()             help on collection methods # 集合~
            sh.help()                    sharding helpers # 分片~
            rs.help()                    replica set helpers # 复制~
            help admin                   administrative help # 管理~
            help connect                 connecting to a db help # 连接~
            help keys                    key shortcuts # 快捷键~
            help misc                    misc things to know # 内置封装类
            help mr                      mapreduce

            show dbs                     show database names
            show collections             show collections in current database
            show users                   show users in current database
            show profile                 show most recent system.profile entries with time >= 1ms
            show logs                    show the accessible logger names
            show log [name]              prints out the last segment of log in memory, 'global' is default
            use <db_name>                set current database
            db.foo.find()                list objects in collection foo
            db.foo.find( { a : 1 } )     list objects in foo where a == 1
            it                           result of the last line evaluated; use to further iterate
            DBQuery.shellBatchSize = x   set default number of items to display on shell
            exit                         quit the mongo shell
    ```

3. 游标相关

    执行find()方法的时候可以用不同的游标方法去修改find()的行为，可以用help()查看可以用那些游标方法

    ![image-20200915184347283](/img/mongodb/image-20200915184347283.png)

### 编写脚本

mongo shell 由JavaScript实现，可以编写JavaScript脚本进行数据处理。

​	创建新连接

```shell
new Mongo()
new Mongo(<host>)
new Mongo(<host:port>)
```

​	选择DB

```shell
conn = new Mongo();
db = conn.getDB("myDatabase");
```

>可以用connect()方法连接并选择DB
>
>```shell
>db = connect("localhost:27020/myDatabase");
>```

​	getDB()和connect()方法会将db设置未全局变量。

常用命令和javascript写法转换：

| Shell Helpers                | JavaScript Equivalents                                       |
| :--------------------------- | :----------------------------------------------------------- |
| `show dbs`, `show databases` | copycopied`db.adminCommand('listDatabases') `                |
| `use <db> `                  | copycopied`db = db.getSiblingDB('<db>') `                    |
| `show collections `          | copycopied`db.getCollectionNames() `                         |
| `show users `                | copycopied`db.getUsers() `                                   |
| `show roles `                | copycopied`db.getRoles({showBuiltinRoles: true}) `           |
| `show log <logname> `        | copycopied`db.adminCommand({ 'getLog' : '<logname>' }) `     |
| `show logs `                 | copycopied`db.adminCommand({ 'getLog' : '*' }) `             |
| `it `                        | copycopied`cursor = db.collection.find() if ( cursor.hasNext() ){   cursor.next(); }` |

--evel 可以向mongo传入JavaScript片段

```shell
mongo test --eval "printjson(db.getCollectionNames())"
```

引入JavaScript脚本

```shell
mongo localhost:27017/test myjsfile.js
```

以上两种是在mongo shell外引入JavaScript 如果是在mongo shell中引入使用lload()方法

```shell
load("myjstest.js")
```

load()中指定js脚本可以是相对路径，也可以是绝对路径

```shell
load("scripts/myjstest.js")
load("/data/db/scripts/myjstest.js")
```

> 相对路径是指相对于当前工作路径而言的；

### 数据类型

1. 数据类型

   Date

    > 带符号的64位整数，表示1970-1-1以来的毫秒数，并不是所有的操作和驱动支持完全的64位建议年份在0~9999之间

    * Date()返回当前时间的字符串；

    * new Date() 返回ISODate()；

    * ISODate 返回 ISODate()；

    ```shell
    > Date()
    Fri Sep 11 2020 09:34:15 GMT+0800
    > ISODate()
    ISODate("2020-09-11T01:35:21.324Z")
    > var date = new Date();
    ISODate("2020-09-11T01:35:34.051Z")
    ```

   ObjectId 

    > 4字节时间戳+5字节随机数+3字节递增计数器

    ```shell
    new ObjectId
    ```

    ```shell
    > var objId = new ObjectId
    > objId
    ObjectId("5f5ade697825049c62468fa8")
    ```

   NumberLong

    >  默认情况下，mongo shell认为所有的数字为浮点数

    NumberLong() 构造器接收字符串参数

    ```shell
    NumberLong("2090845886852")
    ```

    eg.

   ![image-20200915184948876](/img/mongodb/image-20200915184948876.png)

    >  $inc 用浮点数对'calc'进行递增会使其类型变为浮点数
    >
    > ![image-20200915185051155](/img/mongodb/image-20200915185051155.png)

    NumberInt

    > 默认情况下，mongo shell 认为所有的数字为浮点数， NumberInt用来显示构造32位整数

    NumberDecimal

    > mongo shell提供了NumberDecimal（）构造函数来显式指定基于`128位`的，基于十进制的浮点值，该值能够精确地模拟十进制舍入。此功能适用于处理货币数据的应用程序，例如`金融`，`税收`和`科学计算`。

    NumberDecimal() 构造器接收字符串参数

    ```shell
    > NumberDecimal("1000.55")
    NumberDecimal("1000.55")
    ```

    > NumberDecimal() 构造器,也接受double类型的浮点数，和java 一样会有精度缺失的问题。并且存在隐式转换将双精度转换为15位精度
    >
    > ```shell
    > > var number = NumberDecimal(1000.55);
    > > number
    > NumberDecimal("1000.55000000000")
    > ```
    >
    > 精度缺失eg.
    >
    > ```shell
    > > var number = NumberDecimal(9999999.4999999999);
    > > number
    > NumberDecimal("9999999.50000000")
    > ```

2. 比较和排序

   浮点数和其他数字类型进行比较取决于他们的实际值。二进制双精度浮点数的十进制表示只是其近似值不一定完全相等，建议使用NumberDecimal();

   数据集：

   ```shell
   { "_id" : 1, "val" : NumberDecimal( "9.99" ), "description" : "Decimal" }
   { "_id" : 2, "val" : 9.99, "description" : "Double" }
   { "_id" : 3, "val" : 10, "description" : "Double" }
   { "_id" : 4, "val" : NumberLong(10), "description" : "Long" }
   { "_id" : 5, "val" : NumberDecimal( "10.0" ), "description" : "Decimal" }
   ```

   查询：

   | Query                                  | Results                                                      |
   | :------------------------------------- | :----------------------------------------------------------- |
   | { “val”: 9.99 }                        | { “_id”: 2, “val”: 9.99, “description”: “Double” }      |
   | { “val”: NumberDecimal( “9.99” ) }| { “_id”: 1, “val”: NumberDecimal( “9.99” ), “description”: “Decimal” } |
   | { val: 10 }                       | { “_id”: 3, “val”: 10, “description”: “Double” }<br>{ “_id”: 4, “val”: NumberLong(10), “description”: “Long” }<br/>{ “_id”: 5, “val”: NumberDecimal( “10.0” ), “description”: “Decimal” } |
   | { val: NumberDecimal( “10” ) }  | { “_id”: 3, “val”: 10, “description”: “Double” }<br/>{ “_id”: 4, “val”: NumberLong(10), “description”: “Long” }<br/>{ “_id”: 5, “val”: NumberDecimal( “10.0” ), “description”: “Decimal” } |

   十进制9.99对应的实际二进制值是不不等于9.99的因此只等查询到Double类型的一条数据；

   十进制10和二进制下的10实际值是一样的，因此可以查询到Double和其他数字类型。

   > 可以用$type查询特定的数据类型的值
   >
   > ```shell
   > db.inventory.find( { price: { $type: "decimal" } } )
   > ```

3. 数据类型判断：

   instanceof 判断是否为某数据类型 返回true或false

   ```shell
   > number
   NumberDecimal("9999999.50000000")
   > number instanceof NumberDecimal
   true
   > number instanceof NumberLong
   false
   ```
   
   typeof 返回具体的数据类型
   
   ```shell
   > typeof number
   object
   ```
   
   

### 快速参考

1. 历史命令

   > shell 中的上下键，选择历史命令。历史命令放在~/.dbshell文件中。
   
2. 命令行参数

   * --help 查看命令行参数信息
  ![image-20200915185414200](/img/mongodb/image-20200915185414200.png)
   
     ![image-20200915185454123](/img/mongodb/image-20200915185454123.png)
   
     
   
   * --nodb 启动mongo shell,不连接数据库
   
     ```shell
     $ ./mongo.exe --nodb
     MongoDB shell version v4.0.10
     > db # 没连接db
     2020-09-15T16:50:43.941+0800 E QUERY    [js] ReferenceError: db is not defined :
     @(shell):1:1
     ```
   
   * --shell 和JavaScript脚本一起使用，执行完脚本继续使用mongo shell
   
3. 帮助相关方法命令

   * help 查看帮助信息

     ```shell
     > help
             db.help()                    help on db methods
             db.mycoll.help()             help on collection methods
             sh.help()                    sharding helpers
             rs.help()                    replica set helpers
             help admin                   administrative help
             help connect                 connecting to a db help
             help keys                    key shortcuts
             help misc                    misc things to know
             help mr                      mapreduce
     
             show dbs                     show database names
             show collections             show collections in current database
             show users                   show users in current database
             show profile                 show most recent system.profile entries with time >= 1ms
             show logs                    show the accessible logger names
             show log [name]              prints out the last segment of log in memory, 'global' is default
             use <db_name>                set current database
             db.foo.find()                list objects in collection foo
             db.foo.find( { a : 1 } )     list objects in foo where a == 1
             it                           result of the last line evaluated; use to further iterate
             DBQuery.shellBatchSize = x   set default number of items to display on shell
             exit                         quit the mongo shell
     ```

   * db.help() 查看db相关的方法

     ```shell
      db.help()
     DB methods:
             db.adminCommand(nameOrDocument) - switches to 'admin' db, and runs command [just calls db.runCommand(...)]
             db.aggregate([pipeline], {options}) - performs a collectionless aggregation on this database; returns a cursor
             db.auth(username, password) # 登录
             db.cloneDatabase(fromhost) - deprecated
             db.commandHelp(name) returns the help for the command # 命令帮助
             db.copyDatabase(fromdb, todb, fromhost) - deprecated
             db.createCollection(name, {size: ..., capped: ..., max: ...})
             db.createView(name, viewOn, [{$operator: {...}}, ...], {viewOptions})
             db.createUser(userDocument) # 创建用户
             db.currentOp() displays currently executing operations in the db
             db.dropDatabase() # 删除数据库
             ...
     ```

   * db.\<collectionName>.help() 集合相关的方法帮助 可以是存在的集合也可以是不存在的集合

     ```shell
     > show collections
     files
     files.chunks
     files.files
     ```

     ```shell
     > db.files.help()
     DBCollection help
             db.files.find().help() - show DBCursor help
             db.files.bulkWrite( operations, <optional params> ) - bulk execute write operations, optional parameters are: w, wtimeout, j
             db.files.count( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
             db.files.countDocuments( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
             ...
     ```

     ```shell
     > db.f.help()
     DBCollection help
             db.f.find().help() - show DBCursor help
             db.f.bulkWrite( operations, <optional params> ) - bulk execute write operations, optional parameters are: w, wtimeout, j
             db.f.count( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
             db.f.countDocuments( query = {}, <optional params> ) - count the number of documents that matches the query, optional parameters are: limit, skip, hint, maxTimeMS
             db.f.estimatedDocumentCount( <optional params> ) - estimate the document count using collection metadata, optional parameters are: maxTimeMS
             ...
     ```

   * show dbs 展示server上的所有db

     ```shell
     > show dbs
     admin   0.000GB
     config  0.000GB
     files   0.013GB
     local   0.000GB
     test    0.000GB
     ```

   * use \<db> 切换当前db

     ```shell
     > db
     files
     > use test
     switched to db test
     > db
     test
     ```

   * show collections 展示当前db下的所有集合

     ```shell
     > show collections
     files
     files.chunks
     files.files
     ```

   * show users 展示当前库的用户

     ```shell
     > show dbs
     admin   0.000GB
     config  0.000GB
     files   0.013GB
     local   0.000GB
     test    0.000GB
     > use admin
     switched to db admin
     > show users
     {
             "_id" : "admin.root",
             "userId" : UUID("1198568b-e718-4c8c-b91b-e551de9d819c"),
             "user" : "root",
             "db" : "admin",
             "roles" : [
                     {
                             "role" : "root",
                             "db" : "admin"
                     }
             ],
             "mechanisms" : [
                     "SCRAM-SHA-1",
                     "SCRAM-SHA-256"
             ]
     }
     ```

   * show  roles 展示当前库的所有角色包括内置和自定义的

     ```shell
     > show roles
     {
             "role" : "__queryableBackup",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "__system",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "backup",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "clusterAdmin",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "clusterManager",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "clusterMonitor",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "dbAdmin",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "dbAdminAnyDatabase",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "dbOwner",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "enableSharding",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "hostManager",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "read",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "readAnyDatabase",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "readWrite",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "readWriteAnyDatabase",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "restore",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "root",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "userAdmin",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     {
             "role" : "userAdminAnyDatabase",
             "db" : "admin",
             "isBuiltin" : true,
             "roles" : [ ],
             "inheritedRoles" : [ ]
     }
     ```

     

   * show profile 打印5个耗时大于1毫秒的最新的更新操作
   * show databases 打印可用数据库
   * load() 加载脚本文件

   

4. JavaScript 常用方法

   > db 参数是当前数据库，默认是test，可以用use命令切换当前库

   * db.auth() 安全模式下 验证用户
   * coll = db.\<collection>  获取集合
   * db.collection.find() 查询当前集合的所有文档
   * db.collection.insertOne() 当前集合插入一条记录
   * db.collection.updateOne() 当前集合更新一条记录
   * db.collection.updateMany() 当前集合批量更新
   * db.collection.save() 当前集合插入新记录或更新已存在的记录
   * db.collection.deleteOne() 当前集合删除一条记录
   * db.collection.deleteMany() 当前集合删除多条记录
   * db.collection.drop() 删除当前集合
   * db.collection.createIndex() 给当前集合建索引，如果索引已经存在了，则操作无效
   * db.getSiblingDB() 用当前连接去连接另一个db

5. 查询

   * db.collection.find(\<query>)
     查询与\<query>参数相匹配的数据，如果\<query>为空，则返回全部数据。类比SQL中 where子句。

     ```shell
     > db.coll1.find()
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     > db.coll1.find({name:'lisi'})
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     ```

   * db.collection.find(\<query>,\<projection>)

     \<projection>返回字段，类比SQL中的SELECT子句。

     ```shell
     > db.coll1.find({name:'lisi'},{name:true})
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi" }
     > db.coll1.find({name:'lisi'},{age:true})
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "age" : 16 }
     > db.coll1.find({name:'lisi'},{age:true,name:true})
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     ```

   * db.collection.find().sort(\<sort order>)
     结果集排序，sort按照什么字段排序，order取值 1和-1，1正序，-1倒序。

     ```shell
     > db.coll1.find().sort({age:1})
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     > db.coll1.find().sort({age:-1})
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     ```

   * db.collection.find().limit(\<n>) 限制返回结果集的条数

     ```shell
     > db.coll1.find().limit(1)
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     > db.coll1.find().limit(2)
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     ```

   * db.collection.find().skip(\<n>) 跳过几条

     ```shell
     > db.coll1.find()
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     { "_id" : ObjectId("5f608d3bbdfef17fa6413670"), "name" : "wangwu", "age" : 17 }
     > db.coll1.find().skip(1)
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     { "_id" : ObjectId("5f608d3bbdfef17fa6413670"), "name" : "wangwu", "age" : 17 }
     > db.coll1.find().skip(2)
     { "_id" : ObjectId("5f608d3bbdfef17fa6413670"), "name" : "wangwu", "age" : 17 }
     ```

     

   * db.collection.find().count() 返回记录数

     ```shell
     > db.coll1.find()
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     { "_id" : ObjectId("5f608d3bbdfef17fa6413670"), "name" : "wangwu", "age" : 17 }
     > db.coll1.find().count()
     3
     > db.coll1.find({name:'zhangsan'}).count()
     1
     ```

     

   * db.collection.findOne(\<query>) 返回单独一条 不传\<query>则返回第一条

     ```
     > db.coll1.find()
     { "_id" : ObjectId("5f6089c1bdfef17fa641366e"), "name" : "zhangsan", "age" : 15 }
     { "_id" : ObjectId("5f6089cebdfef17fa641366f"), "name" : "lisi", "age" : 16 }
     { "_id" : ObjectId("5f608d3bbdfef17fa6413670"), "name" : "wangwu", "age" : 17 }
     > db.coll1.findOne()
     {
             "_id" : ObjectId("5f6089c1bdfef17fa641366e"),
             "name" : "zhangsan",
             "age" : 15
     }
     > db.coll1.findOne({name:'lisi'})
     {
             "_id" : ObjectId("5f6089cebdfef17fa641366f"),
             "name" : "lisi",
             "age" : 16
     }
     ```

     

   * db.fromCollection.renameCollection(\<toCollectionName>) 修改集合名

     ```shell
     > show collections
     coll1
     collection
     > db.coll
     db.coll1       db.collection
     > db.coll1.renameCollection('coll2')
     { "ok" : 1 }
     > show collections
     coll2
     collection
     ```

     