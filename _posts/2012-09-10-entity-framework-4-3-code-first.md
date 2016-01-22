---
layout: post
title: "Entity Framework 4.3 Code First"
date: 2012-09-10 09:00
author: Jeremy Gray
comments: true
categories: [software]
tags: [C#, Entity Framework, jeremy]
---
This is the third part of a multi-part series.  Links to other posts are listed after the article.

**History**

It seems to me that the Entity Framework is the latest in an attempt by Microsoft to separate people who work in the database from people who work on applications.  Truthfully, I'm not sure how I feel about this...but it's the world in which we live.  The concept is that we create classes in Visual Studio and they magically appear in SQL Server.  This has been a magic unicorn in the computer science field for as long as I can remember.    In fact, EF 4 was called the [magic unicorn edition](http://www.hanselman.com/blog/SimpleCodeFirstWithEntityFramework4MagicUnicornFeatureCTP4.aspx).  Most problems come from what is called [object-relational impedance mismatch](http://en.wikipedia.org/wiki/Object-relational_impedance_mismatch) which has a lot of facets you can read about somewhere else, however most people will get the general idea from the following fictional conversation:

<a href="{{ site.baseurl }}/images/ghostbusters_movie_image_01.jpg">!["Ghostbusters]({{ site.baseurl }}/images/ghostbusters_movie_image_01.jpg)</a>

|  |           |
| -------------: | :----------- |
| **Stanz**      | Fire and brimstone coming down from the sky! Rivers and seas boiling!|
| **Spengler**     | Forty years of darkness! Earthquakes, volcanoes!     |
| **Winston Zeddmore**     | The dead rising from the grave!     |
| **Venkman**     | Human sacrifice! Dogs and cats, living together! *Mass hysteria*!     |


One of my favorite articles about this topic is called [Object-Relational Mapping is the Vietnam of Computer Science](http://www.codinghorror.com/blog/2006/06/object-relational-mapping-is-the-vietnam-of-computer-science.html).

**Getting Started**
At first, I found it a little bit clunky to use the Powershell host in the Package Manager Console to communicate my changes to the database, but over time I got used to it.  Every time you want to change an object you must run the following commands

~~~~~~~~
Add-Migration <identifier>
Update-Database
~~~~~~~~

After the Add-Migration command executes a file is created within your solution containing all the changes that exist in your objects, but not in the database and saves the changes in a class named after your identifier that inherits the abstract class DbMigration.  This class implements [Up()](http://msdn.microsoft.com/en-us/library/system.data.entity.migrations.dbmigration.up(v=vs.103).aspx) and [Down()](http://msdn.microsoft.com/en-us/library/system.data.entity.migrations.dbmigration.down(v=vs.103).aspx) methods that map out exactly how to bridge the application / database barrier in both directions.  This gives us the opportunity to apply migrations in reverse and go to a previous state with one of the following equivalent commands:


~~~~~~~~
Update-Database -TargetMigration:"<identifier>"
Update-Database -TargetMigration:0
Update-Database -TargetMigration:$InitialDatabase
~~~~~~~~


I think that is a pretty big deal.

<a href="{{ site.baseurl }}/images/migrations.png">!["Entity Framework Migrations]({{ site.baseurl }}/images/migrations.png)</a>

The Migrations folder, shown on left, is automatically created and a file is created for every migration.  I don't personally like the idea of a folder that contains an infinite number of files that I have to maintain, but I believe you can delete the migrations and regenerate as one file.

If you want the full experience, I suggest reading the [official documentation](http://msdn.microsoft.com/en-us/data/jj591621).

**Moving from Test to Production**

My final worry with this product is moving between environments, which didn't turn out to be be very difficult.  Adding a -Script argument to the Update Database command will generate a sql file that can be applied to other environments as necessary.  This process was seamless for me but [YMMV](http://www.urbandictionary.com/define.php?term=YMMV).

**Boo, No Enums**
The biggest problem that I had with EF 4.3 Code Migrations is the lack of support for enums.  As we all know an enum is a distinct type consisting of a set of named constants, which, translated into the database world is a lookup or code table.  So I would expect the following two constructs to be equivalent in their respective systems.

~~~~~~~~
enum DaysOfWeek {Sun, Mon, Tue, Wed, Thu, Fri, Sat};
~~~~~~~~

<a href="{{ site.baseurl }}/images/dow11.png">![Days of Week - Database]({{ site.baseurl }}/images/dow11.png)</a>
Days of the Week - Represented in a Database

They are not.  The only way to store enums in the database at this point is to put the text of an enum into the database field.  I recognize developing this feature is a difficult task to accomplish, but was almost a deal-breaker in this project because there were a TON of enums.  By default EF ignores enums forcing me to create another public property on the bject, demonstrated below.

~~~~~~~~
public string MyEnumValue
{
    get { return this.MyEnum.ToString(); }
    set { if (null != value) MyEnum = (MyEnum)Enum.Parse(typeof(MyEnum), value); }
}
~~~~~~~~

I felt like I was going back in time 10 years when I ended up with a column named MyEnumValue that contained a string.  It hurt me.

This has allegedly been addressed in version 5 that released on 8/15/2012, but I have not had time to upgrade.



| **Conclusions** |           |
| :------------- | ----------- |
| I have yet to get used to visualizing database style relationships within Visual Studio but I see myself using this product again in the future.  It gets my stamp of approval.  I think it took a little longer this time but at the end of the day I have a testable project getting pounded in production and have had no issues (related to EF).      | <a href="{{ site.baseurl }}/images/works-on-my-machine.png">![Works on my Machine]({{ site.baseurl }}/images/works-on-my-machine.png)</a>
|

**All Posts in this Series**


1.  [Introduction](http://lotsofgigs.wordpress.com/2012/08/27/windows-service-project-introduction/)
2.  [Where the project ended up](http://lotsofgigs.wordpress.com/2012/09/03/windows-service-project-where-it-ended-up/)
3.  [Entity Framework 4.3](http://lotsofgigs.wordpress.com/2012/09/10/entity-framework-4-3-code-first/)
4.  [Custom File Helpers build + creating Excel documents on windows server 2008](http://lotsofgigs.wordpress.com/2012/09/24/creating-excel-documents-on-windows-server-2008-with-custom-file-helpers-build/)
5.  <del>Problems encountered</del>
** **
