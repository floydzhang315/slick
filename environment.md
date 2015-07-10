# 准备开发环境
本篇介绍如果设置使用 Slick 的 Scala 开发环境，这里我们使用 SBT 命令行，SBT  使用的目录结构和 Maven 一样，我们可以创建一个目录，比如 Slick ，然后创建如下的缺省目录结构：

- src
  - main
      - java
      - resources
      - scala
  - test
  - java
  - resources
  - scala

因为我们打算使用 MySQL 数据库，并使用 Slick 来访问数据库，因此我们在 Slick 的根目录下创建一个 build.sbt，添加相关引用：
```
name := "Scala Slick Examples"

version := "1.0"

scalaVersion := "2.10.4"

libraryDependencies += "com.typesafe.slick" %% "slick" % "2.0.2"

libraryDependencies += "org.slf4j" % "slf4j-nop" % "1.6.4"

libraryDependencies += "mysql" % "mysql-connector-java" % "5.1.18"
```
Slick 使用 SLF4J 作为日志库文件。

我们的 MySQL 数据库 Chinook 安装在本地服务器上面，我们在使用 Slick 可以手工创建数据库表 Schema 的定义，也可以使用自动代码生成工具从已有的数据库创建 Table 的 Schema 定义。

我们在命令行输入 sbt，进入 SBT 控制台。

然后我们使用 console，进入 Scala 控制台，注意此时 SBT 自动把 build.sbt 中引用到的库比如 slick，mysql 添加到 Scala 控制台，我们使用如下命令：
```
scala.slick.model.codegen.SourceCodeGenerator.main(
    Array(slickDriver, jdbcDriver, url, outputFolder, pkg, user, password)
)
```
相关参数如下：
slickDriver Fully qualified name of Slick driver class, e.g. “scala.slick.driver.H2Driver”
jdbcDriver Fully qualified name of jdbc driver class, e.g. “org.h2.Driver”
url jdbc url, e.g. “jdbc:postgresql://localhost/test”
outputFolder Place where the package folder structure should be put
pkg Scala package the generated code should be places in
user database connection user name
password database connection password

例如对于本例，我们使用 mysql 数据库，可以在命令行输入如下命令：注意修改你的用户名和密码：
```
scala.slick.model.codegen.SourceCodeGenerator.main(
    Array("scala.slick.driver.MySQLDriver", "com.mysql.jdbc.Driver", 
    "jdbc:mysql://127.0.0.1/Chinook", 
    "./src/main/scala",
    "com.guidebee.slick.example", "user", "password")
)
```
这样自动代码生成工具，就在 /src/main/scala 目录下生成了 Tables.scala 文件
```
package com.guidebee.slick.example
// AUTO-GENERATED Slick data model
/** Stand-alone Slick data model for immediate use */
object Tables extends {
    val profile = scala.slick.driver.MySQLDriver
} with Tables

/** Slick data model trait for extension, choice of backend or usage in the cake pattern. (Make sure to initialize this late.) */
trait Tables {
    val profile: scala.slick.driver.JdbcProfile
    import profile.simple._
    import scala.slick.model.ForeignKeyAction
    // NOTE: GetResult mappers for plain SQL are only generated for tables where Slick knows how to map the types of all columns.
    import scala.slick.jdbc.{GetResult => GR}
  
    /** DDL for all tables. Call .create to execute. */
    lazy val ddl = Album.ddl ++ Artist.ddl ++ Customer.ddl ++ Employee.ddl ++ Genre.ddl ++ Invoice.ddl ++ Invoiceline.ddl ++ Mediatype.ddl ++ Playlist.ddl ++ Playlisttrack.ddl ++ Track.ddl
  
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
        def ? = (albumid.?, title.?, artistid.?).shaped.<>({r=>import r._; _1.map(_=> AlbumRow.tupled((_1.get, _2.get, _3.get)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
        /** Database column AlbumId PrimaryKey */
        val albumid: Column[Int] = column[Int]("AlbumId", O.PrimaryKey)
        /** Database column Title  */
        val title: Column[String] = column[String]("Title")
        /** Database column ArtistId  */
        val artistid: Column[Int] = column[Int]("ArtistId")
    
        /** Foreign key referencing Artist (database name FK_AlbumArtistId) */
        lazy val artistFk = foreignKey("FK_AlbumArtistId", artistid, Artist)(r => r.artistid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
    }
    /** Collection-like TableQuery object for table Album */
    lazy val Album = new TableQuery(tag => new Album(tag))
  
    /** Entity class storing rows of table Artist
     *  @param artistid Database column ArtistId PrimaryKey
     *  @param name Database column Name  */
    case class ArtistRow(artistid: Int, name: Option[String])
    /** GetResult implicit for fetching ArtistRow objects using plain SQL queries */
    implicit def GetResultArtistRow(implicit e0: GR[Int], e1: GR[Option[String]]): GR[ArtistRow] = GR{
        prs => import prs._
        ArtistRow.tupled((<<[Int], <<?[String]))
    }
    /** Table description of table Artist. Objects of this class serve as prototypes for rows in queries. */
    class Artist(tag: Tag) extends Table[ArtistRow](tag, "Artist") {
        def * = (artistid, name) <> (ArtistRow.tupled, ArtistRow.unapply)
        /** Maps whole row to an option. Useful for outer joins. */
        def ? = (artistid.?, name).shaped.<>({r=>import r._; _1.map(_=> ArtistRow.tupled((_1.get, _2)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
        /** Database column ArtistId PrimaryKey */
        val artistid: Column[Int] = column[Int]("ArtistId", O.PrimaryKey)
        /** Database column Name  */
        val name: Column[Option[String]] = column[Option[String]]("Name")
    }
    /** Collection-like TableQuery object for table Artist */
    lazy val Artist = new TableQuery(tag => new Artist(tag))
  
    /** Entity class storing rows of table Customer
      *  @param customerid Database column CustomerId PrimaryKey
      *  @param firstname Database column FirstName 
      *  @param lastname Database column LastName 
      *  @param company Database column Company 
      *  @param address Database column Address 
      *  @param city Database column City 
      *  @param state Database column State 
      *  @param country Database column Country 
      *  @param postalcode Database column PostalCode 
      *  @param phone Database column Phone 
      *  @param fax Database column Fax 
      *  @param email Database column Email 
      *  @param supportrepid Database column SupportRepId  */
    case class CustomerRow(customerid: Int, firstname: String, lastname: String, company: Option[String], address: Option[String], city: Option[String], state: Option[String], country: Option[String], postalcode: Option[String], phone: Option[String], fax: Option[String], email: String, supportrepid: Option[Int])
    /** GetResult implicit for fetching CustomerRow objects using plain SQL queries */
    implicit def GetResultCustomerRow(implicit e0: GR[Int], e1: GR[String], e2: GR[Option[String]], e3: GR[Option[Int]]): GR[CustomerRow] = GR{
    prs => import prs._
    CustomerRow.tupled((<<[Int], <<[String], <<[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<[String], <<?[Int]))
    }
  /** Table description of table Customer. Objects of this class serve as prototypes for rows in queries. */
class Customer(tag: Tag) extends Table[CustomerRow](tag, "Customer") {
    def * = (customerid, firstname, lastname, company, address, city, state, country, postalcode, phone, fax, email, supportrepid) <> (CustomerRow.tupled, CustomerRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (customerid.?, firstname.?, lastname.?, company, address, city, state, country, postalcode, phone, fax, email.?, supportrepid).shaped.<>({r=>import r._; _1.map(_=> CustomerRow.tupled((_1.get, _2.get, _3.get, _4, _5, _6, _7, _8, _9, _10, _11, _12.get, _13)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column CustomerId PrimaryKey */
    val customerid: Column[Int] = column[Int]("CustomerId", O.PrimaryKey)
    /** Database column FirstName  */
    val firstname: Column[String] = column[String]("FirstName")
    /** Database column LastName  */
    val lastname: Column[String] = column[String]("LastName")
    /** Database column Company  */
    val company: Column[Option[String]] = column[Option[String]]("Company")
    /** Database column Address  */
    val address: Column[Option[String]] = column[Option[String]]("Address")
    /** Database column City  */
    val city: Column[Option[String]] = column[Option[String]]("City")
    /** Database column State  */
    val state: Column[Option[String]] = column[Option[String]]("State")
    /** Database column Country  */
    val country: Column[Option[String]] = column[Option[String]]("Country")
    /** Database column PostalCode  */
    val postalcode: Column[Option[String]] = column[Option[String]]("PostalCode")
    /** Database column Phone  */
    val phone: Column[Option[String]] = column[Option[String]]("Phone")
    /** Database column Fax  */
    val fax: Column[Option[String]] = column[Option[String]]("Fax")
    /** Database column Email  */
    val email: Column[String] = column[String]("Email")
    /** Database column SupportRepId  */
    val supportrepid: Column[Option[Int]] = column[Option[Int]]("SupportRepId")
    
    /** Foreign key referencing Employee (database name FK_CustomerSupportRepId) */
    lazy val employeeFk = foreignKey("FK_CustomerSupportRepId", supportrepid, Employee)(r => r.employeeid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
}
    /** Collection-like TableQuery object for table Customer */
    lazy val Customer = new TableQuery(tag => new Customer(tag))
  
  /** Entity class storing rows of table Employee
   *  @param employeeid Database column EmployeeId PrimaryKey
   *  @param lastname Database column LastName 
   *  @param firstname Database column FirstName 
   *  @param title Database column Title 
   *  @param reportsto Database column ReportsTo 
   *  @param birthdate Database column BirthDate 
   *  @param hiredate Database column HireDate 
   *  @param address Database column Address 
   *  @param city Database column City 
   *  @param state Database column State 
   *  @param country Database column Country 
   *  @param postalcode Database column PostalCode 
   *  @param phone Database column Phone 
   *  @param fax Database column Fax 
   *  @param email Database column Email  */
    case class EmployeeRow(employeeid: Int, lastname: String, firstname: String, title: Option[String], reportsto: Option[Int], birthdate: Option1, hiredate: Option1, address: Option[String], city: Option[String], state: Option[String], country: Option[String], postalcode: Option[String], phone: Option[String], fax: Option[String], email: Option[String])
  /** GetResult implicit for fetching EmployeeRow objects using plain SQL queries */
    implicit def GetResultEmployeeRow(implicit e0: GR[Int], e1: GR[String], e2: GR[Option[String]], e3: GR[Option[Int]], e4: GR[Option1]): GR[EmployeeRow] = GR{
    prs => import prs._
    EmployeeRow.tupled((<<[Int], <<[String], <<[String], <<?[String], <<?[Int], <<?1, <<?1, <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<?[String]))
}
/** Table description of table Employee. Objects of this class serve as prototypes for rows in queries. */
class Employee(tag: Tag) extends Table[EmployeeRow](tag, "Employee") {
    def * = (employeeid, lastname, firstname, title, reportsto, birthdate, hiredate, address, city, state, country, postalcode, phone, fax, email) <> (EmployeeRow.tupled, EmployeeRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (employeeid.?, lastname.?, firstname.?, title, reportsto, birthdate, hiredate, address, city, state, country, postalcode, phone, fax, email).shaped.<>({r=>import r._; _1.map(_=> EmployeeRow.tupled((_1.get, _2.get, _3.get, _4, _5, _6, _7, _8, _9, _10, _11, _12, _13, _14, _15)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column EmployeeId PrimaryKey */
    val employeeid: Column[Int] = column[Int]("EmployeeId", O.PrimaryKey)
    /** Database column LastName  */
    val lastname: Column[String] = column[String]("LastName")
    /** Database column FirstName  */
    val firstname: Column[String] = column[String]("FirstName")
    /** Database column Title  */
    val title: Column[Option[String]] = column[Option[String]]("Title")
    /** Database column ReportsTo  */
    val reportsto: Column[Option[Int]] = column[Option[Int]]("ReportsTo")
    /** Database column BirthDate  */
    val birthdate: Column[Option1] = column[Option1]("BirthDate")
    /** Database column HireDate  */
    val hiredate: Column[Option1] = column[Option1]("HireDate")
    /** Database column Address  */
    val address: Column[Option[String]] = column[Option[String]]("Address")
    /** Database column City  */
    val city: Column[Option[String]] = column[Option[String]]("City")
    /** Database column State  */
    val state: Column[Option[String]] = column[Option[String]]("State")
    /** Database column Country  */
    val country: Column[Option[String]] = column[Option[String]]("Country")
    /** Database column PostalCode  */
    val postalcode: Column[Option[String]] = column[Option[String]]("PostalCode")
    /** Database column Phone  */
    val phone: Column[Option[String]] = column[Option[String]]("Phone")
    /** Database column Fax  */
    val fax: Column[Option[String]] = column[Option[String]]("Fax")
    /** Database column Email  */
    val email: Column[Option[String]] = column[Option[String]]("Email")
    
    /** Foreign key referencing Employee (database name FK_EmployeeReportsTo) */
    lazy val employeeFk = foreignKey("FK_EmployeeReportsTo", reportsto, Employee)(r => r.employeeid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
  }
  /** Collection-like TableQuery object for table Employee */
  lazy val Employee = new TableQuery(tag => new Employee(tag))
  
  /** Entity class storing rows of table Genre
   *  @param genreid Database column GenreId PrimaryKey
   *  @param name Database column Name  */
  case class GenreRow(genreid: Int, name: Option[String])
  /** GetResult implicit for fetching GenreRow objects using plain SQL queries */
  implicit def GetResultGenreRow(implicit e0: GR[Int], e1: GR[Option[String]]): GR[GenreRow] = GR{
    prs => import prs._
    GenreRow.tupled((<<[Int], <<?[String]))
  }
  /** Table description of table Genre. Objects of this class serve as prototypes for rows in queries. */
  class Genre(tag: Tag) extends Table[GenreRow](tag, "Genre") {
    def * = (genreid, name) <> (GenreRow.tupled, GenreRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (genreid.?, name).shaped.<>({r=>import r._; _1.map(_=> GenreRow.tupled((_1.get, _2)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column GenreId PrimaryKey */
    val genreid: Column[Int] = column[Int]("GenreId", O.PrimaryKey)
    /** Database column Name  */
    val name: Column[Option[String]] = column[Option[String]]("Name")
  }
  /** Collection-like TableQuery object for table Genre */
  lazy val Genre = new TableQuery(tag => new Genre(tag))
  
  /** Entity class storing rows of table Invoice
   *  @param invoiceid Database column InvoiceId PrimaryKey
   *  @param customerid Database column CustomerId 
   *  @param invoicedate Database column InvoiceDate 
   *  @param billingaddress Database column BillingAddress 
   *  @param billingcity Database column BillingCity 
   *  @param billingstate Database column BillingState 
   *  @param billingcountry Database column BillingCountry 
   *  @param billingpostalcode Database column BillingPostalCode 
   *  @param total Database column Total  */
  case class InvoiceRow(invoiceid: Int, customerid: Int, invoicedate: java.sql.Timestamp, billingaddress: Option[String], billingcity: Option[String], billingstate: Option[String], billingcountry: Option[String], billingpostalcode: Option[String], total: scala.math.BigDecimal)
  /** GetResult implicit for fetching InvoiceRow objects using plain SQL queries */
  implicit def GetResultInvoiceRow(implicit e0: GR[Int], e1: GR1, e2: GR[Option[String]], e3: GR1): GR[InvoiceRow] = GR{
    prs => import prs._
    InvoiceRow.tupled((<<[Int], <<[Int], <<1, <<?[String], <<?[String], <<?[String], <<?[String], <<?[String], <<1))
  }
  /** Table description of table Invoice. Objects of this class serve as prototypes for rows in queries. */
  class Invoice(tag: Tag) extends Table[InvoiceRow](tag, "Invoice") {
    def * = (invoiceid, customerid, invoicedate, billingaddress, billingcity, billingstate, billingcountry, billingpostalcode, total) <> (InvoiceRow.tupled, InvoiceRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (invoiceid.?, customerid.?, invoicedate.?, billingaddress, billingcity, billingstate, billingcountry, billingpostalcode, total.?).shaped.<>({r=>import r._; _1.map(_=> InvoiceRow.tupled((_1.get, _2.get, _3.get, _4, _5, _6, _7, _8, _9.get)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column InvoiceId PrimaryKey */
    val invoiceid: Column[Int] = column[Int]("InvoiceId", O.PrimaryKey)
    /** Database column CustomerId  */
    val customerid: Column[Int] = column[Int]("CustomerId")
    /** Database column InvoiceDate  */
    val invoicedate: Column1 = column1("InvoiceDate")
    /** Database column BillingAddress  */
    val billingaddress: Column[Option[String]] = column[Option[String]]("BillingAddress")
    /** Database column BillingCity  */
    val billingcity: Column[Option[String]] = column[Option[String]]("BillingCity")
    /** Database column BillingState  */
    val billingstate: Column[Option[String]] = column[Option[String]]("BillingState")
    /** Database column BillingCountry  */
    val billingcountry: Column[Option[String]] = column[Option[String]]("BillingCountry")
    /** Database column BillingPostalCode  */
    val billingpostalcode: Column[Option[String]] = column[Option[String]]("BillingPostalCode")
    /** Database column Total  */
    val total: Column1 = column1("Total")
    
    /** Foreign key referencing Customer (database name FK_InvoiceCustomerId) */
    lazy val customerFk = foreignKey("FK_InvoiceCustomerId", customerid, Customer)(r => r.customerid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
  }
  /** Collection-like TableQuery object for table Invoice */
  lazy val Invoice = new TableQuery(tag => new Invoice(tag))
  
  /** Entity class storing rows of table Invoiceline
   *  @param invoicelineid Database column InvoiceLineId PrimaryKey
   *  @param invoiceid Database column InvoiceId 
   *  @param trackid Database column TrackId 
   *  @param unitprice Database column UnitPrice 
   *  @param quantity Database column Quantity  */
  case class InvoicelineRow(invoicelineid: Int, invoiceid: Int, trackid: Int, unitprice: scala.math.BigDecimal, quantity: Int)
  /** GetResult implicit for fetching InvoicelineRow objects using plain SQL queries */
  implicit def GetResultInvoicelineRow(implicit e0: GR[Int], e1: GR1): GR[InvoicelineRow] = GR{
    prs => import prs._
    InvoicelineRow.tupled((<<[Int], <<[Int], <<[Int], <<1, <<[Int]))
  }
  /** Table description of table InvoiceLine. Objects of this class serve as prototypes for rows in queries. */
  class Invoiceline(tag: Tag) extends Table[InvoicelineRow](tag, "InvoiceLine") {
    def * = (invoicelineid, invoiceid, trackid, unitprice, quantity) <> (InvoicelineRow.tupled, InvoicelineRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (invoicelineid.?, invoiceid.?, trackid.?, unitprice.?, quantity.?).shaped.<>({r=>import r._; _1.map(_=> InvoicelineRow.tupled((_1.get, _2.get, _3.get, _4.get, _5.get)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column InvoiceLineId PrimaryKey */
    val invoicelineid: Column[Int] = column[Int]("InvoiceLineId", O.PrimaryKey)
    /** Database column InvoiceId  */
    val invoiceid: Column[Int] = column[Int]("InvoiceId")
    /** Database column TrackId  */
    val trackid: Column[Int] = column[Int]("TrackId")
    /** Database column UnitPrice  */
    val unitprice: Column1 = column1("UnitPrice")
    /** Database column Quantity  */
    val quantity: Column[Int] = column[Int]("Quantity")
    
    /** Foreign key referencing Invoice (database name FK_InvoiceLineInvoiceId) */
    lazy val invoiceFk = foreignKey("FK_InvoiceLineInvoiceId", invoiceid, Invoice)(r => r.invoiceid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
    /** Foreign key referencing Track (database name FK_InvoiceLineTrackId) */
    lazy val trackFk = foreignKey("FK_InvoiceLineTrackId", trackid, Track)(r => r.trackid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
  }
  /** Collection-like TableQuery object for table Invoiceline */
  lazy val Invoiceline = new TableQuery(tag => new Invoiceline(tag))
  
  /** Entity class storing rows of table Mediatype
   *  @param mediatypeid Database column MediaTypeId PrimaryKey
   *  @param name Database column Name  */
  case class MediatypeRow(mediatypeid: Int, name: Option[String])
  /** GetResult implicit for fetching MediatypeRow objects using plain SQL queries */
  implicit def GetResultMediatypeRow(implicit e0: GR[Int], e1: GR[Option[String]]): GR[MediatypeRow] = GR{
    prs => import prs._
    MediatypeRow.tupled((<<[Int], <<?[String]))
  }
  /** Table description of table MediaType. Objects of this class serve as prototypes for rows in queries. */
  class Mediatype(tag: Tag) extends Table[MediatypeRow](tag, "MediaType") {
    def * = (mediatypeid, name) <> (MediatypeRow.tupled, MediatypeRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (mediatypeid.?, name).shaped.<>({r=>import r._; _1.map(_=> MediatypeRow.tupled((_1.get, _2)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column MediaTypeId PrimaryKey */
    val mediatypeid: Column[Int] = column[Int]("MediaTypeId", O.PrimaryKey)
    /** Database column Name  */
    val name: Column[Option[String]] = column[Option[String]]("Name")
  }
  /** Collection-like TableQuery object for table Mediatype */
  lazy val Mediatype = new TableQuery(tag => new Mediatype(tag))
  
  /** Entity class storing rows of table Playlist
   *  @param playlistid Database column PlaylistId PrimaryKey
   *  @param name Database column Name  */
  case class PlaylistRow(playlistid: Int, name: Option[String])
  /** GetResult implicit for fetching PlaylistRow objects using plain SQL queries */
  implicit def GetResultPlaylistRow(implicit e0: GR[Int], e1: GR[Option[String]]): GR[PlaylistRow] = GR{
    prs => import prs._
    PlaylistRow.tupled((<<[Int], <<?[String]))
  }
  /** Table description of table Playlist. Objects of this class serve as prototypes for rows in queries. */
  class Playlist(tag: Tag) extends Table[PlaylistRow](tag, "Playlist") {
    def * = (playlistid, name) <> (PlaylistRow.tupled, PlaylistRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (playlistid.?, name).shaped.<>({r=>import r._; _1.map(_=> PlaylistRow.tupled((_1.get, _2)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column PlaylistId PrimaryKey */
    val playlistid: Column[Int] = column[Int]("PlaylistId", O.PrimaryKey)
    /** Database column Name  */
    val name: Column[Option[String]] = column[Option[String]]("Name")
  }
  /** Collection-like TableQuery object for table Playlist */
  lazy val Playlist = new TableQuery(tag => new Playlist(tag))
  
  /** Entity class storing rows of table Playlisttrack
   *  @param playlistid Database column PlaylistId 
   *  @param trackid Database column TrackId  */
  case class PlaylisttrackRow(playlistid: Int, trackid: Int)
  /** GetResult implicit for fetching PlaylisttrackRow objects using plain SQL queries */
  implicit def GetResultPlaylisttrackRow(implicit e0: GR[Int]): GR[PlaylisttrackRow] = GR{
    prs => import prs._
    PlaylisttrackRow.tupled((<<[Int], <<[Int]))
  }
  /** Table description of table PlaylistTrack. Objects of this class serve as prototypes for rows in queries. */
  class Playlisttrack(tag: Tag) extends Table[PlaylisttrackRow](tag, "PlaylistTrack") {
    def * = (playlistid, trackid) <> (PlaylisttrackRow.tupled, PlaylisttrackRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (playlistid.?, trackid.?).shaped.<>({r=>import r._; _1.map(_=> PlaylisttrackRow.tupled((_1.get, _2.get)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column PlaylistId  */
    val playlistid: Column[Int] = column[Int]("PlaylistId")
    /** Database column TrackId  */
    val trackid: Column[Int] = column[Int]("TrackId")
    
    /** Primary key of Playlisttrack (database name PlaylistTrack_PK) */
    val pk = primaryKey("PlaylistTrack_PK", (playlistid, trackid))
    
    /** Foreign key referencing Playlist (database name FK_PlaylistTrackPlaylistId) */
    lazy val playlistFk = foreignKey("FK_PlaylistTrackPlaylistId", playlistid, Playlist)(r => r.playlistid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
    /** Foreign key referencing Track (database name FK_PlaylistTrackTrackId) */
    lazy val trackFk = foreignKey("FK_PlaylistTrackTrackId", trackid, Track)(r => r.trackid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
  }
  /** Collection-like TableQuery object for table Playlisttrack */
  lazy val Playlisttrack = new TableQuery(tag => new Playlisttrack(tag))
  
  /** Entity class storing rows of table Track
   *  @param trackid Database column TrackId PrimaryKey
   *  @param name Database column Name 
   *  @param albumid Database column AlbumId 
   *  @param mediatypeid Database column MediaTypeId 
   *  @param genreid Database column GenreId 
   *  @param composer Database column Composer 
   *  @param milliseconds Database column Milliseconds 
   *  @param bytes Database column Bytes 
   *  @param unitprice Database column UnitPrice  */
  case class TrackRow(trackid: Int, name: String, albumid: Option[Int], mediatypeid: Int, genreid: Option[Int], composer: Option[String], milliseconds: Int, bytes: Option[Int], unitprice: scala.math.BigDecimal)
  /** GetResult implicit for fetching TrackRow objects using plain SQL queries */
  implicit def GetResultTrackRow(implicit e0: GR[Int], e1: GR[String], e2: GR[Option[Int]], e3: GR[Option[String]], e4: GR1): GR[TrackRow] = GR{
    prs => import prs._
    TrackRow.tupled((<<[Int], <<[String], <<?[Int], <<[Int], <<?[Int], <<?[String], <<[Int], <<?[Int], <<1))
  }
  /** Table description of table Track. Objects of this class serve as prototypes for rows in queries. */
  class Track(tag: Tag) extends Table[TrackRow](tag, "Track") {
    def * = (trackid, name, albumid, mediatypeid, genreid, composer, milliseconds, bytes, unitprice) <> (TrackRow.tupled, TrackRow.unapply)
    /** Maps whole row to an option. Useful for outer joins. */
    def ? = (trackid.?, name.?, albumid, mediatypeid.?, genreid, composer, milliseconds.?, bytes, unitprice.?).shaped.<>({r=>import r._; _1.map(_=> TrackRow.tupled((_1.get, _2.get, _3, _4.get, _5, _6, _7.get, _8, _9.get)))}, (_:Any) =>  throw new Exception("Inserting into ? projection not supported."))
    
    /** Database column TrackId PrimaryKey */
    val trackid: Column[Int] = column[Int]("TrackId", O.PrimaryKey)
    /** Database column Name  */
    val name: Column[String] = column[String]("Name")
    /** Database column AlbumId  */
    val albumid: Column[Option[Int]] = column[Option[Int]]("AlbumId")
    /** Database column MediaTypeId  */
    val mediatypeid: Column[Int] = column[Int]("MediaTypeId")
    /** Database column GenreId  */
    val genreid: Column[Option[Int]] = column[Option[Int]]("GenreId")
    /** Database column Composer  */
    val composer: Column[Option[String]] = column[Option[String]]("Composer")
    /** Database column Milliseconds  */
    val milliseconds: Column[Int] = column[Int]("Milliseconds")
    /** Database column Bytes  */
    val bytes: Column[Option[Int]] = column[Option[Int]]("Bytes")
    /** Database column UnitPrice  */
    val unitprice: Column1 = column1("UnitPrice")
    
    /** Foreign key referencing Album (database name FK_TrackAlbumId) */
    lazy val albumFk = foreignKey("FK_TrackAlbumId", albumid, Album)(r => r.albumid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
    /** Foreign key referencing Genre (database name FK_TrackGenreId) */
    lazy val genreFk = foreignKey("FK_TrackGenreId", genreid, Genre)(r => r.genreid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
    /** Foreign key referencing Mediatype (database name FK_TrackMediaTypeId) */
    lazy val mediatypeFk = foreignKey("FK_TrackMediaTypeId", mediatypeid, Mediatype)(r => r.mediatypeid, onUpdate=ForeignKeyAction.NoAction, onDelete=ForeignKeyAction.NoAction)
    }
    /** Collection-like TableQuery object for table Track */
    lazy val Track = new TableQuery(tag => new Track(tag))
}
```
