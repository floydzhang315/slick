# 数据库 Schema
我们之前 Slick 编程(2): 准备开发环境使用自动代码生成工具生成数据库表的 Slick 定义（使用 Lifted Embedding API)，本篇介绍如何手工来写这些 Schema 定义。

**数据库表 Tables**  
为了能够使用 Slick 的 Lifted Embedding API 定义类型安全的查询，首先我们需要定义数据库表代表表中每行数据的类和对应于数据库表的 Schema 的 TableQuery 值，我们先看看自动生成的 Album 表个相关定义：
```
/** Entity class storing rows of table Album
   *  @param albumid Database column AlbumId PrimaryKey
   *  @param title Database column Title 
   *  @param artistid Database column ArtistId  */
  case class AlbumRow(albumid: Int, title: String, artistid: Int)
  /** GetResult implicit for fetching AlbumRow objects using plain SQL queries */
  implicit def GetResultAlbumRow(implicit e0: GR[Int], e1: GR[String]): GR[AlbumRow] = GR{
    prs => import prs._
    AlbumRow.tupled((<<[Int], <<[String], <<[Int]))
  }
  /** Table description of table Album. Objects of this class serve as prototypes for rows in queries. */
  class Album(tag: Tag) extends Table[AlbumRow](tag, "Album") {
    def * = (albumid, title, artistid) <> (AlbumRow.tupled, AlbumRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (albumid.?, title.?, artistid.?).shaped.<>(
		{r=>import r._; _1.map(_=> AlbumRow.tupled((_1.get, _2.get, _3.get)))}, 
		(_:Any) =>  throw new Exception("Inserting into ? projection not supported."))

    /** Database column AlbumId PrimaryKey */
    val albumid: Column[Int] = column[Int]("AlbumId", O.PrimaryKey)
    /** Database column Title  */
    val title: Column[String] = column[String]("Title")
    /** Database column ArtistId  */
    val artistid: Column[Int] = column[Int]("ArtistId")

    /** Foreign key referencing Artist (database name FK_AlbumArtistId) */
    lazy val artistFk = foreignKey("FK_AlbumArtistId", artistid, Artist)
		(r => r.artistid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
  }
  /** Collection-like TableQuery object for table Album */
  lazy val Album = new TableQuery(tag => new Album(tag))
```
所有的字段(Column)使用 column 方法来定义，每个字段对应一个 Scala 类型和一个字段名称（对应到数据库表的定义），下面为 Slick 支持的基本数据类型：

 

 - Numeric types: Byte, Short, Int, Long, BigDecimal, Float, Double
 - LOB types: java.sql.Blob, java.sql.Clob, Array[Byte]
 - Date types: java.sql.Date, java.sql.Time, java.sql.Timestamp
 - Boolean
 - String
 - Unit
 - java.util.UUID

支持 Null 的字段使用 Option[T] 来表示，其中 T 为上述基本数据类型，在字段名称之后，你可以使用一些可选的字段定义，这些可选定义定义在 table 的O对象中。下面为常用的定义

PrimaryKey 表明该字段为主键
Default[T](defaultValue: T) 该字段缺省值
DBType(dbType: String) 非标准字段类型，比如 DBType(“VARCHAR(20)”) 做为 String 类型
AutoInc 自动增一的字段
NotNull, Nullable 表明该字段是否可以为空

每个表定义都需要一个"*"方法定义了缺省映射，这定义了执行查询返回表格一行时的数据类型，Slick 的”*”不要求和数据库表的定义一一映射，你可以添加字段（复合字段）或者省略掉某个字段。

**匹配过的表定义**  
可以使用自定义的数据类型做为”*”的映射，这可以使用双向映射操作符”<>“来完成。
比如：
```
def * = (albumid, title, artistid) <> (AlbumRow.tupled, AlbumRow.unapply)
```
**约束**  
外键约束可以使用foreignKey来定义
```
/** Foreign key referencing Artist (database name FK_AlbumArtistId) */
    lazy val artistFk = foreignKey("FK_AlbumArtistId", artistid, Artist)
		(r => r.artistid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
```
它的参数为外键约束的名称，本表字段名称，外键所在表名称，和一个函数，这个函数定义了外键约束，以及更新和删除外键时的行为）

主键约束可以使用 primaryKey 来定义，这主要用作定义复合主键的情况
```
/** Primary key of Playlisttrack (database name PlaylistTrack_PK) */
    val pk = primaryKey("PlaylistTrack_PK", (playlistid, trackid))
```
其它比如索引的情况和主键约束非常类似，比如：
```
class A(tag: Tag) extends Table[(Int, Int)](tag, "a") {
    def k1 = column[Int]("k1")
    def k2 = column[Int]("k2")
    def * = (k1, k2)
    def idx = index("idx_a", (k1, k2), unique = true)
    // compiles to SQL:
    //   create unique index "idx_a" on "a" ("k1","k2")
}
```
**数据库定义语言 DDL**  
数据库定义语句可以使用 TableQuery 的 ddl 方法，多个 DDL 对象可以使用 ++ 连接，
比如：
```
val ddl = coffees.ddl ++ suppliers.ddl
db withDynSession {
    ddl.create
    //...
    ddl.drop
}
```
ddl.create 和 ddl.drop 可以创建表和删除表，如果需要看看对应的 SQL 语句，可以使用
```
val ddl = Album.ddl
ddl.createStatements.foreach(println)
ddl.dropStatements.foreach(println)
```
对应的 MySQL 语句为
```
create table `Album` (`AlbumId` INTEGER NOT NULL PRIMARY KEY,`Title` VARCHAR(254) NOT NULL,`ArtistId` INTEGER NOT NULL)
alter table `Album` add constraint `FK_AlbumArtistId` foreign key(`ArtistId`) references `Artist`(`ArtistId`) on update NO ACTION on delete NO ACTION
ALTER TABLE Album DROP FOREIGN KEY FK_AlbumArtistId
drop table `Album`
```