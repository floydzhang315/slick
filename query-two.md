# 查询（二）
**Union**
两个查询的结果可以通过 ++ （或者 unionAll ) 和 union 操作联合起来：
```
val q1= Album.filter(_.artistid <10)
val q2 = Album.filter(_.artistid > 15)
val unionQuery  = q1 union q2
val unionAllQuery = q1 ++ q2
```
union 操作会去掉重复的结果，而 unionAll 只是简单的把两个查询结果连接起来（通常来说比较高效）。
**Aggregation**
和 SQL 一样，Slick 也有 min，max，sum，avg 等集合操作
```
val q = Album.map(_.artistid)
val q1 = q.max
val q2 = q.min 
val q3 = q.avg 
val q4 = q.sum

```
注意：这里 q.max，min，avg，sum 返回结果类型为 Column[Option[T]]，要得到最好的 scalar 类型的值 T，可以调用 run，得到 Option[T]，然后再调用 Option 的 get 或 getOrDefault，
比如：
```
val q = Album.map(_.artistid)
val q1 = q.max 
println(q1.run.get)
```
得到打印的结果：
275

其它的 Aggregation 操作还有 length，exists，比如：
```
val q1 = Album.length
val q2 = Album.exists
```
分组使用 groupBy 操作，类似于 Scala 集合类型的 groupBy 操作：
```
val q= (for {
	 a <- Album
	 b <- Artist
	 if a.artistid === b.artistid
   } yield (b.artistid,b.name)
).groupBy(_._2)
val q1 = q.map { case (name, records) =>
		(records.map(_._1).avg, name,records.length)}
q1 foreach println 

```
这段代码使用两个查询，给出 Album 根据艺术家出的专辑的统计，其中中间查询 q，包含一个嵌套的 Query，目前 Scala 不支持直接查询嵌套的 Query，因此我们需要分两次查询，打印出的部分结果如下：
```
(Some(230),Some(Aaron Copland & London Symphony Orchestra),1)
(Some(202),Some(Aaron Goldberg),1)
(Some(1),Some(AC/DC),2)
(Some(214),Some(Academy of St. Martin in the Fields & Sir Neville Marriner),1)
(Some(215),Some(Academy of St. Martin in the Fields Chamber Ensemble & Sir Neville Marriner),1)
(Some(222),Some(Academy of St. Martin in the Fields, John Birch, Sir Neville Marriner & Sylvia McNair),1)
(Some(257),Some(Academy of St. Martin in the Fields, Sir Neville Marriner & Thurston Dart),1)
(Some(2),Some(Accept),2)
(Some(260),Some(Adrian Leaper & Doreen de Feis),1)
(Some(3),Some(Aerosmith),1)
(Some(197),Some(Aisha Duo),1)
(Some(4),Some(Alanis Morissette),1)
(Some(206),Some(Alberto Turco & Nova Schola Gregoriana),1)
(Some(5),Some(Alice In Chains),1)
(Some(252),Some(Amy Winehouse),2)
...
```