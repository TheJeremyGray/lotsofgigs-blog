---
layout: archivepost
title: "ILDASM to the Rescue!"
date: 2012-07-02 13:16
author: Jeremy Gray
comments: true
categories: [software]
tags: [C#, DevExpress 12.1 Upgrade, Disassembler, ILDASM, software]
---
So we recently upgraded our [DevExpress](http://www.devexpress.com/) control suite to 12.1.4.0, which was a big deal for us because it requires .NET 3.5, which was not installed on many of the PC's across our global organization.  I know what you are thinking,


>"The [.NET framework 3.5](http://en.wikipedia.org/wiki/.NET_Framework_3.5#.NET_Framework_3.5) was released in 2007...how is it not available in your organization?"


I can't answer this question.  It is now.

Since many users across our organization do not have administrative access to their computers, we design all our windows applications to be "installed" with a simple file copy.  Which means deploying all dependencies along with the application.  This generally isn't an issue but at certain times when our controls vendor, DevExpress, sends out a major update we cherry pick about 40 MB of the 182 MB in dependencies to send across the wire to every client.  When this happens we make a best guess about which dependencies we need, deploy them, and test.  It's usually seamless.  So I was surprised to see the following error pop up this morning.


>System.TypeLoadException: Method 'get_LayoutUnit' in type 'DevExpress.XtraRichEdit.RichEditControl' from assembly 'DevExpress.XtraRichEdit.v12.1, Version=12.1.4.0, Culture=neutral, PublicKeyToken=b88d1754d700e49a' does not have an implementation.


<a href="http://lotsofgigs.files.wordpress.com/2012/07/capture.png">![](http://lotsofgigs.files.wordpress.com/2012/07/capture.png?w=300 "Error from Test Machine")</a>

I couldn't believe it, but was able to replicate on a test machine.  The errors we usually get are that DLL's are not found, not "Method has no implementation"

So I did a quick dis-assembly of the DLL with [ILDASM](http://msdn.microsoft.com/en-us/library/f7dy01k1(v=vs.80).aspx) and found that a new dependency was introduced in this version: DevExpress.Office.v12.1.Core.  Code below.


>ildasm /text/ /output="c:\test.txt" "c:\pathToBinary\DevExpress.XtraRichEdit.v12.1.dll"

~~~~~~~~
.method public hidebysig newslot specialname virtual final
instance valuetype ['**DevExpress.Office.v12.1.Core**']DevExpress.XtraRichEdit.DocumentLayoutUnit
get_LayoutUnit() cil managed
{
...
IL_000e: callvirt instance valuetype ['**DevExpress.Office.v12.1.Core**']DevExpress.XtraRichEdit.DocumentLayoutUnit ['DevExpress.RichEdit.v12.1.Core']DevExpress.XtraRichEdit.Internal.InnerRichEditDocumentServer::get_LayoutUnit()
...
} // end of method RichEditControl::get_LayoutUnit
~~~~~~~~

I deployed this new assembly and voilà! the exception is no more.

I did a little research and found [some tools](http://www.codeproject.com/Articles/246858/Depends4Net-Part-1) that could help me diagnose these issues and found a few more new dependencies that we have overlooked.
