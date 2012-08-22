---
layout: post
title: "Adding file size in bytes to Explorer's Details View on Windows 2000 and XP"
date: 2012-08-22 01:19
comments: true
categories: 
---

Mi biggest pet peeve on Windows Explorer is that there is no way to show the size of a file in bytes without having to open the [properties dialog box](https://raw.github.com/dredix/FileSizeXP/master/doc/images/FileProperties.png). You would think the Details view is a great place for that, considering the remarkable selection of columns you can include. Seriously, there is a _Mood_ column but no _Size in bytes_ or equivalent?

On Windows XP and previous versions, you could write a [shell extension](http://msdn.microsoft.com/en-us/library/windows/desktop/cc144067.aspx) that implemented the [IColumnProvider](http://msdn.microsoft.com/en-us/library/windows/desktop/bb776147.aspx) interface and add custom columns as needed. [TortoiseSVN](http://tortoisesvn.net/docs/release/TortoiseSVN_en/tsvn-dug-wcstatus.html#id663427) used these kind of extensions, for example, to show the status of files under source control directly on Windows Explorer.

On Windows Vista, however, Microsoft [decided to remove the IColumnProvider interface](http://msdn.microsoft.com/en-us/library/windows/desktop/bb776147.aspx):

> **Note**  Support for **IColumnProvider** has been removed as of Windows Vista. The Windows property system is used in its place. See [Windows Property System](http://msdn.microsoft.com/en-us/library/windows/desktop/ff728898.aspx) for conceptual materials that explain the use of the new system.

My guess is that rogue extensions were causing a lot of performance/stability issues on Windows Explorer and Microsoft didn't like being blamed for them ("Why is Explorer so slow? Stupid Microsoft Die Die Die!"), so they decided to retire the interface altogether. I haven't got too far into the Windows property system but I suspect you cannot mimic the full functionality of custom columns on the details view for every type of file.

Anyway, even if I came 11 years late to the party and it's no longer relevant, I decided to take a shot at it and implemented a [shell extension for showing the size of file in bytes](https://github.com/dredix/FileSizeXP). No more [undesirable rounding up/down to kilobytes](http://blogs.msdn.com/b/oldnewthing/archive/2011/03/15/10140985.aspx) for the benefit of ancient programs that have probably fallen into oblivion.

When installed, the extension adds a new column called __Size in Bytes__ which can be added to the explorer's Details View:

<a href="https://lh5.googleusercontent.com/-Eq_XZFBzVcg/UDQ8w4Z1gfI/AAAAAAAAAIw/XdMl1-_HoZQ/s800/ColumnSelection.png"><img alt="Column Selection" src="https://lh5.googleusercontent.com/-Eq_XZFBzVcg/UDQ8w4Z1gfI/AAAAAAAAAIw/XdMl1-_HoZQ/s800/ColumnSelection.png" /></a>

Select it, and you'll get an extra column with the file size in glorious bytes:

<a href="https://lh3.googleusercontent.com/-DgKb-qm1ByA/UDQ8wyqjbOI/AAAAAAAAAI4/NKNUQbOrAeg/s800/Explorer.png"><img alt="Details View" style="width: 100%" src="https://lh3.googleusercontent.com/-DgKb-qm1ByA/UDQ8wyqjbOI/AAAAAAAAAI4/NKNUQbOrAeg/s800/Explorer.png" /></a>

If you still use Windows 2000 or XP <s>don't you think is time to move on?</s> and want to use the extension, follow these steps:

* Download the DLL: [FileSizeXP.dll](https://github.com/dredix/FileSizeXP/raw/master/bin/FileSizeXP.dll).

* Put it wherever you want. C:\Utils is as good a place as any.

* Register it by running the following command:  
    <code>regsvr32 C:\path\to\file\FileSizeXP.dll</code>

* Restart the PC.

Note, however, the following caveats:

* The extension does not, I repeat, DOES NOT work on Windows Vista and newer versions. For that you will need need an Explorer replacement, like [Zabkat's Xplorer<sup>2</sup>](http://zabkat.com/), or just give up on showing full size in bytes in Explorer.

* I haven't actually tried the extension on Windows 2000.

* Sorting files by the new column seems to be broken at the moment. I'll go back and fix it, hopefully before Windows XP goes out of support.

If you decided to trust a random guy on the Internet and try this extension, please do let me know how it went.

