---
layout: archivepost
title: "Creating Excel documents on Windows Server 2008 with Custom File Helpers Build"
date: 2012-09-24 09:09
author: Jeremy Gray
comments: true
categories: [software]
tags: [C#, excel, fail, jeremy, software, windows server 2008]
---
This is the fourth and final part of a multi-part series.  Links to other posts are listed after the article.

![](http://lh5.ggpht.com/_2VEaTPMR9yw/TBttmQzYD0I/AAAAAAAAAhw/hb6KUWZa6zk/cat_fail%5B2%5D.jpg "Fail")

Let me start with a lovely quote from the [Microsoft support website](http://support.microsoft.com/kb/257757).

`Microsoft does not currently recommend, and does not support, Automation of Microsoft Office applications from any unattended, non-interactive client application or component (including ASP, ASP.NET, DCOM, and NT Services), because Office may exhibit unstable behavior and/or deadlock when Office is run in this environment.`

So I decided to create an unattended, non-interactive application sitting on Windows Server 2008.  I made this decision for a few reasons,


1.  We generate a LOT of "Excel" files daily for transmission to third parties.  They all want Excel...not CSV, not XLSB, not SLYK, not XML, not DIF, not PDF/XPS.  If Excel "barks" at them at all about file format or Excel doesn't automatically recognize the file type I get a phone call.  I hate getting these phone calls.  [[Sidebar](http://en.wikipedia.org/wiki/Sidebar_(law)): If CSV had bold, we could get away with it but sending a million dollar invoice with no formatting is out of the question.]
2.  We have 67 different ways to create an "Excel" document and they are all very different.
3.  Regardless of being a good practice or not, I routinely get people asking me to pull data from an Excel spreadsheet and put it in a different format or insert into a database.
4.  It just didn't seem like that big of a deal.

**I think this was the biggest mistake of my career.**

<a href="http://i40.tinypic.com/2lm32ae.jpg">![Grumpy Cat is Grumpy](http://i40.tinypic.com/2lm32ae.jpg)</a>

It didn't cost a lot of time or money, but it was classic "When all you have is a hammer, everything looks like a nail".  This project gave me an excuse to create a solution that we could use in the rest of our applications that would solve some big problems, which I like.  I didn't take into account that every call to my application would spin up Excel via COM, do some stuff, then tear it down.  This is an expensive operation and if I used it to create every document we needed, it would most likely take down the server that it is on.  You just can't have 1000 copies of Excel running on a server and I can't justify adding more web servers to the cluster just so I can create documents.  I didn't look at the big picture, I had experience with Excel in File Helpers and decided to double-down.

**So here's how I did it, you've been warned**

A few months ago I open-sourced a version of FileHelpers that had some significant changes to the [Excel Storage](https://github.com/TheJeremyGray/FileWatcherService/blob/master/FileHelpersLib/FileHelpers.ExcelStorage/ExcelStorage.cs) project.  Namely, I made it implement IDisposable and not create a COM connection on every command.  To me it seemed the author thought  people would be happy enough to write an array of objects to native excel format and wouldn't want to do anything else.  But I needed to do several operations to the Excel document and didn't want to wait to set-up and tear-down the COM connection on every command.  Turned out this was a good  decision several months prior to my project because I needed to do a lot of formatting for the new report I was generating.

I added some public methods that change background color, merge cells, change page orientation, remove sheets, and other common tasks.  Then, ran it on my machine successfully.  Since I had it "working", I promised the project and feature.

But I promised before I ran a successful test on the server...it didn't work on the server, I was in big trouble.

<a href="http://i49.tinypic.com/9gj24i.jpg">![Tiny Cat Needs Help](http://i49.tinypic.com/9gj24i.jpg)</a>


It took a long time to get this working, I'm not sure exactly how long but it was at least an eight hour day.  I probably tried 30 different approaches that I found in different corners of the internet.  Finally, I re-read the support page and a single line jumped out

*They assume an interactive desktop and user profile*

After I re-read this I [flashed](http://en.wikipedia.org/wiki/The_Intersect#.22Flashes.22) on something from earlier in the day (week?) about Excel COM solutions failing because of missing directories. At the time I thought it was preposterous.  Turns out it wasn't.  The following code fixed my problem.

*DirectoryInfo InteropDesktopHack = new DirectoryInfo(@"C:\Windows\SysWOW64\config\systemprofile\Desktop");
if (!InteropDesktopHack.Exists) InteropDesktopHack.Create();*


So I was able to successfully implement automation of Excel via COM Interop on Windows Server 2008, but I will never do it again and as soon as I get a chance I am going to throw what I have done in the trash and replace it with something else.

I'll leave you with a quote from [John Saunders](http://stackoverflow.com/users/76337/john-saunders), emphasis mine

*Let me be more adamant than others have been: **do not use Excel server-side**. It is intended to be used as a desktop application, meaning it is not intended to be used from random different threads, possibly multiple threads at a time. **You're better off writing your own spreadsheet than trying to use Excel from a server**.*

**All Posts in this Series**

1.  [Introduction](http://lotsofgigs.wordpress.com/2012/08/27/windows-service-project-introduction/)
2.  [Where the project ended up](http://lotsofgigs.wordpress.com/2012/09/03/windows-service-project-where-it-ended-up/)
3.  [Entity Framework 4.3](http://lotsofgigs.wordpress.com/2012/09/10/entity-framework-4-3-code-first/)
4.  [Custom File Helpers build + creating Excel documents on windows server 2008](http://lotsofgigs.wordpress.com/2012/09/24/creating-excel-documents-on-windows-server-2008-with-custom-file-helpers-build/)
5.  <del>Problems encountered</del>
