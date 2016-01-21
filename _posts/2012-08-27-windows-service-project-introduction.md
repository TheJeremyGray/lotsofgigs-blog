---
layout: post
title: "Windows Service Project - Introduction"
date: 2012-08-27 09:00
author: lotsofgigs
comments: true
categories: [software]
tags: [C#, jeremy, opensource, software, Windows Service]
---

I've been working on a windows service off and on for about a month.  Like every project, it started pretty simple and grew into a much different beast.  In the beginning I was pretty sure where the project would end up so I was planning on the work explosion the whole time.  This will be a multi-part post, so [bear](http://en.wikipedia.org/wiki/Bear) with me.

**Business Case**

My company decided that we wanted to implement a system to facilitate a "fixed price auction".  We wanted to provide a platform for buyers and sellers of different financial instruments to log in and trade non-exchange driven products...like electricity or [uranium](http://en.wikipedia.org/wiki/Enriched_uranium).  We have experts on staff that will set the price, then run an auction for a set amount of time (< 10 min), we match up the buyers and sellers, then run through our existing process of selling non-exchange driven products.

**The Plan**

We wrote the specification and sent the work to a company overseas who specializes in auctions.  The only real part of this process I was involved in was designing the algorithm to allocate products at the end of the auction and importing the results of the auction into our existing system.  I'd love to write another post about the allocation algorithm but would need to talk to my boss first and I doubt it's going to happen.

**My Part**

After the close of the auction, allocations are made and a series of files get sent to me.  Initially, it was  a single Excel spreadsheet with a worksheet containing all the bids and a worksheet containing the winners.  My initial requirement was, "Pick up this Excel Spreadsheet, send the first workbook to this group of people and the second workbook to a different group of people.  Under no circumstances can Group 2 see the info that is for Group 1".  Sounds pretty easy...and for the most part it was.  I even [open-sourced](https://github.com/TheJeremyGray/FileWatcherService) a stripped down version of it.  I wasn't 100% happy with how it turned out, but it worked.  This [commit](https://github.com/TheJeremyGray/FileWatcherService/blob/master/FileHelpersLib/FileHelpers.ExcelStorage/ExcelStorage.cs) was my first official attempt to get code into an open-source library.

**The Service**

I'd forgotten how complex windows service programming *looks*.  Even a [hello world](http://en.wikipedia.org/wiki/Hello_world_program) service takes a minimum of 4 different projects.


1.  *Service Project* - Creates the object windows recognizes as a service.  In my opinion, the OnStart() method should have a single task: create an instance of an object in your class library.  The OnStop() should dispose of objects and whatever components you are using should directly call the class library that holds business logic.
2.  *Class Library* - All business logic should be here, the goal is to create a testable class.
3.  *Console App* - This app mimics the code in the service project and is the start-up project in Visual Studio.  If you press play, your project should just work as a console app and you will be able to debug.
4.  *Setup project* - Creates an .msi to install the service on your remote machine.  If you don't do this you have to run a script similar to [this](https://github.com/TheJeremyGray/FileWatcherService/blob/master/installNETservice.bat) to get the project to show up in the windows services console, then [another script](https://github.com/TheJeremyGray/FileWatcherService/blob/master/uninstallNETservice.bat) to remove it.  The MSI is much easier, you will thank me later.

Of course you could do it with less but good luck.  I added a unit testing project and three [filehelpers](http://filehelpers.com/) libraries, bringing the grand total of projects in the solution to eight.

The only strange thing you will notice if you look at the source code is our corporate use of Google Documents to store information.  In my opinion, this project is a great example of when to use cloud based configuration exposed to a select set of administrators.  My initial requirement was, of course, only to send notifications to certain people but I knew that list would change over time and I didn't want to change it.  So the distribution lists are stored in a [Google Spreadsheet](http://www.google.com/google-d-s/spreadsheets/).  My only other viable option would be to create a table in the database, then prop up a webpage where people could add and remove email addresses off the list.  Since we already have a [Google Apps for Business](http://www.google.com/enterprise/apps/business/) account setup and users trained on how to access the information, the decision was easy.

I have a lot to say about this project: it was fun, challenging, and didn't take a whole lot of hours.  So stay tuned for the next few installments.

**All Posts in this Series**

1.  [Introduction](http://lotsofgigs.wordpress.com/2012/08/27/windows-service-project-introduction/)
2.  [Where the project ended up](http://lotsofgigs.wordpress.com/2012/09/03/windows-service-project-where-it-ended-up/)
3.  [Entity Framework 4.3](http://lotsofgigs.wordpress.com/2012/09/10/entity-framework-4-3-code-first/)
4.  [Custom File Helpers build + creating Excel documents on windows server 2008](http://lotsofgigs.wordpress.com/2012/09/24/creating-excel-documents-on-windows-server-2008-with-custom-file-helpers-build/)
5.  <del>Problems encountered</del>
