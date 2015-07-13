# 数据库连接和事务处理  
你可以在程序的任何地方使用数据库查询，当执行查询时你需要有一个数据库连接。
你可以通过创建一个 Database 对象来连接一个 JDBC 数据库，有多种方法可以创建一个数据库对象。
**使用 JDBC URL**  
你可以使用 JDBC URL 来创建一个 Database 对象（URL 的格式取决于连接的数据库的类型），
比如：

```
val db = Database.forURL("jdbc:h2:mem:test1;DB_CLOSE_DELAY=-1", driver="org.h2.Driver")
```
创建一个基于内存的 H2 数据库连接，再比如我们之前使用的 MySQL 数据库，可以使用
```
val db = Database.forURL("jdbc:mysql://127.0.0.1/Chinook",
    driver = "com.mysql.jdbc.Driver",
    user="user",
    password="password")
```
**使用DataSource**  
你可以使用已有的 datasource 对象，来构建一个 Database 对象，比如你从连接池中取得一个 Datasource 对象，然后连接到 Slick 库中
```
val db = Database.forDataSource(dataSource: javax.sql.DataSource)
```
之后你创建一个 Session 对象，将从连接池中取得一个数据库连接，当关闭 Session 时，连接退回给连接池以作他用。

**使用JNDI 名称**  
如果你使用 JNDI，你可以提供 JNDI 名称来构建一个 Database 对象：
```
val db = Database.forName(jndiName: String)
```
**Session管理**  
现在你有了一个数据库对象可以打开一个数据库（Slick 函数库封装了一个 Session 对象）

Database 的 withSession 方法，创建一个 Session 对象，它可以传递给一个函数，函数返回时自动关闭这个 Session 对象，如果你使用连接池，关闭 Session 对象，自动将连接退回连接池。
```
val query = for (c <- coffees) yield c.name
val result = db.withSession {
    session =>
    query.list()( session )
}
```
你可以看到，我们可以在 withSession 之外定义查询，只有在实际执行查询时才需要一个 Session 对象，要注意的是 Session 的缺省模式为自动提交（auto-commit )模式。每个数据库指令（比如 insert )都自动提交给数据库。 如果需要将几个指令作为一个整体，那么就需要使用事务处理（Transaction）
上面的例子，我们在执行查询时，明确指明了 session 对象，你可以使用隐含对象来避免这种情况，比如：
```
val query = for (c <- coffees) yield c.name
val result = db.withSession {
    implicit session =>
    query.list // <- takes session implicitly
}
// query.list // <- would not compile, no implicit value of type Session
```
**手工管理 Session**  
这不是推荐使用的情况，但如果你需要自己管理 Session 对象，你可以自己管理 Session 的生命周期：
```
val query = for (c <- coffees) yield c.name
val session : Session = db.createSession
val result  = query.list()( session )
session.close
```
**事务处理**  
你可以使用 Session 对象的 withTransaction 方法来创建一个事务，传给该方法的语句将作为一个整体事务执行，如果出现异常，Slick 自动回滚事务，你也可以使用 rollback 强制事务回滚，要注意的是 Slick 只会回滚数据库相关操作，而不会取消其它 Scala 语句。
```
session.withTransaction {
    // your queries go here

    if (/* some failure */ false){
        session.rollback // signals Slick to rollback later
    }

} //
```
如果你没有 Session 对象，也可以直接使用数据库对象的 withTransaction 方法，如：
```
db.withTransaction{
    implicit session =>
    // your queries go here
}
```