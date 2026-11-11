## 用 Node.js 跑通第一个 SQLite 数据库



SQLite 是什么

SQLite 是一个"内嵌式"的 SQL 数据库：它没有独立运行的服务器程序，整个数据库就是硬盘上的一个普通文件，程序可以直接读写这个文件。

从 Node.js v22.5.0 开始，Node 自带了一个 `node:sqlite` 模块，因此不需要额外安装任何数据库驱动包就能操作 SQLite。





1、搭建项目并跑通第一个数据库

```
node --version
```

只要输出的版本号 **≥ v22.5.0** 就可以往下走。



2、用 pnpm 初始化一个项目

```
mkdir practice
cd practice
pnpm init
```

这会生成一个 `package.json`，用来记录这个项目的信息。



3、新建一个脚本文件 `hello-sqlite.mjs`，写入下面的内容：

```js
import { DatabaseSync } from 'node:sqlite'; // 引入`node:sqlite` 模块

// 打开(或创建)一个数据库文件。SQLite 的数据库就是一个普通文件。
const db = new DatabaseSync('data/app.db');

// 如果表notes不存在就创建它。
// id: integer 主键、primary key 主键，content Text 内容字段。
db.exec(`
  CREATE TABLE IF NOT EXISTS notes (
    id INTEGER PRIMARY KEY, 
    content TEXT
  )
`);

// 插入一条新记录。
// INSERT INTO notes — 目标表是 notes。
// (content) — 指定要写入的列，这里只写 content 列。
// VALUES ('Hello, SQLite!') — 对应给出的值，content 列的值是字符串 'Hello, SQLite!'。
db.exec(`INSERT INTO notes (content) VALUES ('Hello, SQLite!')`);


// 查询表里的所有记录。
// SELECT * FROM notes — 从 notes 表中获取所有列的所有记录。
// all() — 获取所有查询结果，返回一个数组。
const rows = db.prepare('SELECT * FROM notes').all();
console.log(rows);

db.close(); // 关闭数据库连接。

```



4、运行它

```
node hello-sqlite.mjs
```

输出（这是本机实测的真实结果）：

```
[ [Object: null prototype] { id: 1, content: 'Hello, SQLite!' } ]
```

查询结果：一个数组，里面一个对象，对应表里的那一行数据。



5、再运行一次，见证"持久化"

```
node hello-sqlite.mjs
```

再看输出：

```
[
  [Object: null prototype] { id: 1, content: 'Hello, SQLite!' },
  [Object: null prototype] { id: 2, content: 'Hello, SQLite!' }
]
```

> 第二次运行，表里多了一条记录，`id` 变成了 2——这说明第一次插入的数据被真实地保存在了 `data/app.db` 这个文件里，而不是随着程序结束就消失了。这就是数据库和普通变量的核心区别：数据能"活"得比程序运行的那一瞬间更久。