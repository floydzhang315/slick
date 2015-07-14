# 查询（三）  
Slick 的查询实际上是执行由 Invoker（无参数时为 UnitInvoker ) Trait 定义的方法，Slick 定义了一个从 Query 隐含的变换,使得你可以直接执行查询操作，最常用的一个情况是把整个查询结果存放到一个 Scala 集合类型中（比如使用 list 方法）
```
val l = q.list
val v = q.buildColl[Vector]
val invoker = q.invoker
val statement = q.selectStatement
```
所有的查询方法都定义了一个隐含参数 Session，如果你愿意，你也可以直接传入一个 session 参数：
```
val l = q.list()(session)
```
如果你只需要单个查询结果，你可以使用 first 或 firstOption 方法，而方法 foreach， foldLeft 和 elements 方法可以用来遍历查询结果而不需要先把结果复制到另外一个 Scala 集合对象中。

**Deleting**  
删除数据和查询很类似，你首先写一个选择查询，然后调用它的 delete 方法，同样 Slick 也定义一个从 Query 到 DeleteInvoker 的隐含转换，DeleteInvoker 定义了 delete 方法
```
val affectedRowsCount = q.delete
val invoker = q.deleteInvoker
val statement = q.deleteStatement
```
定义用来删除记录的查询时只能使用单个表格。

**Inserting**  
插入操作基于单个表定义的字段映射，当你直接使用某个表来插入数据时，这个操作基于表类型中定义的“*”，如果你省略某些字段，那么插入这些省略的字段会使用缺省值，所有的插入操作方法定义在 InsertInvoker 和 FullInsertInvoker。
```
coffees += ("Colombian", 101, 7.99, 0, 0)

coffees ++= Seq(
  ("French_Roast", 49, 8.99, 0, 0),
  ("Espresso",    150, 9.99, 0, 0)
)

// "sales" and "total" will use the default value 0:
coffees.map(c => (c.name, c.supID, c.price)) += ("Colombian_Decaf", 101, 8.99)

val statement = coffees.insertStatement
val invoker = coffees.insertInvoker

// compiles to SQL:
// INSERT INTO "COFFEES" ("COF_NAME","SUP_ID","PRICE","SALES","TOTAL") VALUES (?,?,?,?,?)
```
如果你的插入操作定义了自动增一的字段，该字段会自动忽略，由数据库本身来插入该字段的值。缺省情况 += 返回受影响的行数（通常总为 1），而 ++ 操作给出总计的行数（以 Option 类型给出），你可以使用 returning 修改返回的值，比如返回插入的行的主键：
```
val userId =
  (users returning users.map(_.id)) += User(None, "Stefan", "Zeiger")
```
要注意的是很多数据库只支持返回自动增一的作为主键的那个字段，如果想返回其它字段，可能会抛出 SlickException 异常。

除了上面的插入记录的方法，还可以使用服务器端表达式的方发插入数据： 
```
class Users2(tag: Tag) extends Table[(Int, String)](tag, "users2") {
    def id = column[Int]("id", O.PrimaryKey)
    def name = column[String]("name")
    def * = (id, name)
}
val users2 = TableQuery[Users2]

users2.ddl.create

users2 insert (users.map { u => (u.id, u.first ++ " " ++ u.last) })

users2 insertExpr (users.length + 1, "admin")
```
**Updating**  
更新记录也是先写查询，然后调用 update 方法，比如：
```
val q = for { c <- coffees if c.name === "Espresso" } yield c.price
q.update(10.49)

val statement = q.updateStatement
val invoker = q.updateInvoker
```
update 方法定义在 UpdateInvoker Trait 中。

**Compiled Queries**  
数据库查询时，通常需要定义一些查询参数，比如根据 ID 查找对应的记录。你可以定义一个带参数的函数来定义查询对象，但每次调用该函数时都要重新编译这个查询语句，系统消耗有些大，Slick 支持预编译这个带参数的查询函数，例如：
```
def userNameByIDRange(min: Column[Int], max: Column[Int]) =
  for {
    u <- users if u.id >= min && u.id < max
  } yield u.first

val userNameByIDRangeCompiled = Compiled(userNameByIDRange _)

// The query will be compiled only once:
val names1 = userNameByIDRangeCompiled(2, 5).run
val names2 = userNameByIDRangeCompiled(1, 3).run
```
这种方法支持查询，更新和删除数据。