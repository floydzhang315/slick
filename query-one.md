# 查询（一）
本篇介绍 Slick 的基本查询，比如选择，插入，更新，删除记录等。

**排序和过滤**
Slick 提供了多种方法可以用来排序和过滤，比如：
```
val q = Album.filter(_.albumid === 101)

//select `AlbumId`, `Title`, `ArtistId` 
//from `Album` where `AlbumId` = 101


val q = Album.drop(10).take(5)
//select .`AlbumId` as `AlbumId`, .`Title` as `Title`,
// .`ArtistId` as `ArtistId` from `Album`  limit 10,5


val q = Album.sortBy(_.title.desc)
//select `AlbumId`, `Title`, `ArtistId` 
//from `Album` order by `Title` desc
```
**Join 和 Zipping**
Join 指多表查询，可以有两种不同的方法来实现多表查询，一种是通过明确调用支持多表连接的方法（比如 innerJoin 方法）返回一个多元组，另外一种为隐含连接( implicit join )，它不直接使用这些连接方法（比如 LeftJoin 方法）。

一个隐含的 cross-Join 为 Query 的 flatMap 操作（在 for 表达式中使用多个生成式），例如：
```
val q = for{a <- Album
			b <- Artist
		} yield( a.title, b.name)

//select x2.`Title`, x3.`Name` from `Album` x2, `Artist` x3
```
如果添加一个条件过滤表达式，它就变成隐含的 inner join，例如：
```
val q = for{a <- Album
			b <- Artist
		    if a.artistid === b.artistid
		} yield( a.title, b.name)

//select x2.`Title`, x3.`Name` from `Album` x2, `Artist` x3 
//where x2.`ArtistId` = x3.`ArtistId`

```
明确的多表连接则使用 innerJoin，leftJoin，rightJoin，outerJoin 方法，例如：
```
val explicitCrossJoin = = for {
			 (a,b) <- Album innerJoin Artist  
			 } yield( a.title, b.name)


//select x2.x3, x4.x5 from (select x6.`Title` as x3 from `Album` x6) 
//x2 inner join (select x7.`Name` as x5 from `Artist` x7) x4 on 1=1


val explicitInnerJoin  = for {
		 (a,b) <- Album innerJoin Artist on (_.artistid === _.artistid)
		 } yield( a.title, b.name)
//select x2.x3, x4.x5 from (select x6.`Title` as x3, x6.`ArtistId` as x7 from `Album` x6) x2 
//inner join (select x8.`ArtistId` as x9, x8.`Name` as x5 from `Artist` x8) x4 on x2.x7 = x4.x9


val explicitLeftOuterJoin   = for {
		 (a,b) <- Album leftJoin Artist on (_.artistid === _.artistid)
		 } yield( a.title, b.name.?)
//select x2.x3, x4.x5 from (select x6.`Title` as x3, x6.`ArtistId` as x7 from `Album` x6) x2 
//left outer join (select x8.`ArtistId` as x9, x8.`Name` as x5 from `Artist` x8) x4 on x2.x7 = x4.x9


val explicitRightOuterJoin   = for {
		 (a,b) <- Album rightJoin Artist on (_.artistid === _.artistid)
		 } yield( a.title.?, b.name)
//select x2.x3, x4.x5 from (select x6.`Title` as x3, x6.`ArtistId` as x7 from `Album` x6) x2 
//right outer join (select x8.`ArtistId` as x9, x8.`Name` as x5 from `Artist` x8) x4 on x2.x7 = x4.x9

```
注意 leftJoin 和 rightJoin 中的 b.name.? 和 a.title.? 的”.?” 这是因为外部查询时会产生额外的 NULL 值，你必须保证返回 Option 类型的值。
除了通常的 InnerJoin，LeftJoin，RightJoin 之外，Scala 还提供了 Zip 方法，它的语法类似于 Scala 的集合类型，比如： 
```
val zipJoinQuery  = for {
	   (a,b) <- Album zip Artist
	 } yield( a.title.?, b.name)

```
此外，还有一个 zipWithIndex，可以把一个表的行和一个从 0 开始的整数序列 Zip 操作，相当于给行添加序号，比如
```
val zipWithIndexJoin  = for {
	   (a,idx) <- Album.zipWithIndex 
	 } yield( a.title, idx)
```