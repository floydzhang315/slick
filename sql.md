# 直接使用 SQL 语句  
如果你有需要直接使用 SQL 语句，Slick 也支持你直接使用 SQL 语句。
首先你需要引入一些引用包：
```
import scala.slick.jdbc.{GetResult, StaticQuery => Q}
import scala.slick.jdbc.JdbcBackend.Database
import Q.interpolation
```
其中最重要的一个相关类似 StaticQuery，为简洁起见，我们使用 Q 作为它的别名。连接数据库还是和以前一样 [Slick 编程(4): 数据库连接和事务处理](database-and-transaction-processing.md)

**DDL 和 DML 语句**  
StaticQuery 的方法 updateNA，（NA 代表无参数），它返回 DDL 指令影响的行数，比如使用 H2 数据库

连接数据库：
```
case class Supplier(id:Int, name:String, street:String, city:String, state:String, zip:String)
case class Coffee(name:String, supID:Int, price:Double, sales:Int, total:Int)

Database.forURL("jdbc:h2:mem:test1", driver = "org.h2.Driver") withDynSession {

}
```
创建数据库表：
```
// Create the tables, including primary and foreign keys
Q.updateNA("create table suppliers("+
  "id int not null primary key, "+
  "name varchar not null, "+
  "street varchar not null, "+
  "city varchar not null, "+
  "state varchar not null, "+
  "zip varchar not null)").execute
Q.updateNA("create table coffees("+
  "name varchar not null, "+
  "sup_id int not null, "+
  "price double not null, "+
  "sales int not null, "+
  "total int not null, "+
  "foreign key(sup_id) references suppliers(id))").execute
```
你可以使用字符串和一个 StaticQuery 对象相加（+）构成一个新的 StaticQuery 对象，一个简单的方法是使用 Q.u 和一个字符串相加，Q.u 代表一个相当与 StaticQuery.updateNA(“”)

例如我们在表中插入一些数据：
```
// Insert some suppliers
(Q.u + "insert into suppliers values(101, 'Acme, Inc.', '99 Market Street', 'Groundsville', 'CA', '95199')").execute
(Q.u + "insert into suppliers values(49, 'Superior Coffee', '1 Party Place', 'Mendocino', 'CA', '95460')").execute
(Q.u + "insert into suppliers values(150, 'The High Ground', '100 Coffee Lane', 'Meadows', 'CA', '93966')").execute
```
在 SQL 查询语句中使用字面量不是一种推荐的方法，尤其是当用户提供数据时（不十分安全）， 此时你可以使用 +? 操作符为查询语句绑定一个参数，比如：
```
def insert(c: Coffee) = (Q.u + "insert into coffees values (" +? c.name +
  "," +? c.supID + "," +? c.price + "," +? c.sales + "," +? c.total + ")").execute

// Insert some coffees
Seq(
  Coffee("Colombian", 101, 7.99, 0, 0),
  Coffee("French_Roast", 49, 8.99, 0, 0),
  Coffee("Espresso", 150, 9.99, 0, 0),
  Coffee("Colombian_Decaf", 101, 8.99, 0, 0),
  Coffee("French_Roast_Decaf", 49, 9.99, 0, 0)
).foreach(insert)
```
这段代码相对于 insert into coffees values (?,?,?,?,?)

**查询语句**  
和 updateNA 类似， StaticQuery 还有一个 queryNA 方法，它支持一个类型参数（代表表的一行），比如：
```
Q.queryNA[AlbumRow]("select * from Album") foreach { a => 
	println(" " + a.albumid + " " + a.title + " " + a.artistid)
}
```
这段代码之所以能工作，是因为 Tables.scala 中定义了
```
/** GetResult implicit for fetching AlbumRow objects using plain SQL queries */
implicit def GetResultAlbumRow(implicit e0: GR[Int], e1: GR[String]): GR[AlbumRow] = GR{
    prs => import prs._
    AlbumRow.tupled((<<[Int], <<[String], <<[Int]))
}

```
定义了从 JDBC 类型到 GetResult[T] 的隐含转换，GetResult[T] 为函数 PositionedResult => T 的一个封装。<<[T] 返回指定位置上期望的值。

和 queryNA 对应的带参数的 query 定义了两个类型参数，一个是参数的类型，另外一个是返回的结果的每行的类型，例如：
```
val q2 = Q.query[Int,(Int,String,Int)] ( """
 select albumid,title,artistid from Album where artistid < ?
""")
val l2 = q2.list(10)
for(t <- l2) println( " " + t._1 + " " + t._2 + " " + t._3)
```
返回结果如下：
```
1 For Those About To Rock We Salute You 1
4 Let There Be Rock 1
2 Balls to the Wall 2
3 Restless and Wild 2
5 Big Ones 3
6 Jagged Little Pill 4
7 Facelift 5
8 Warner 25 Anos 6
34 Chill: Brazil (Disc 2) 6
9 Plays Metallica By Four Cellos 7
10 Audioslave 8
11 Out Of Exile 8
271 Revelations 8
12 BackBeat Soundtrack 9

```
Q.interpolation 支持字符串插值，比如：
```
def albumByTitle(title: String) = sql"select * from Album where title = $title".as[AlbumRow]
println("Album: " + albumByTitle("Let There Be Rock").firstOption)
```

使用 sql 做为前缀的字符串，可以将以 $ 开始的变量替换成该变量的值，此外对于 update/delete 语句，可以使用 sqlu 前缀。