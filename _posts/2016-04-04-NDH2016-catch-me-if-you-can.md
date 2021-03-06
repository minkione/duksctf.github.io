---
layout: post
title: "Nuit du hack quals 2016 - Catch me if you can"
date: 2016-04-04
---

*The task is a usb pcap where two files were transfered. The trick was to take
each odd packet number and take 0x708 of each to create the first file, use the
even for the 2nd file. We were left with two Libreoffice ods file. In the 2nd
one, we found a table and a weird alphabetic suite which used against the table
give us  the flag.*

<!--more-->

### Description

*We managed to infect the computer of a target. We recorded all packets
transferred over the USB port, but there is something unusual. We need them to
be sorted to get the juicy secret.*


### Details

Points:         100

Category:       forensic

Validations:    50

### Solution

We were given a file called [usb.pcap](/resources/2016/ndh/catch_me_if_you_can/usb.pcap).
After digging around the file for a while it appears that it's a USB transfer of
several files.

We wrote a simple [python script](/resources/2016/ndh/catch_me_if_you_can/extract_files.py) to extract the different blob with [scapy](http://www.secdev.org/projects/scapy/).

``` python
#!/usr/bin/env python2

from scapy.all import *

pcap = rdpcap("usb.pcap")
for i,p in enumerate(pcap):
	if len(p) > 100:
		open(str(i),"wb").write(p.load[27:])
```

After analyzing those files, we found that there is **two** files in the transfer.
To reconstruct the two files, we simply use odd and even files for each. Here is
the [python script](/resources/2016/ndh/catch_me_if_you_can/prepare_file.py) to do it:


{% highlight python %} #!/usr/bin/env python2
from os.path import join
from os import listdir

working_dir = "working"

folder = []
files1 = []
files2 = []
for i in listdir(working_dir):
    folder.append(i)

folder.sort()
for i in folder:
    if int(i) % 2:
        files2.append(i)
    else:
        files1.append(i)

#reorder the blob
files1.sort()
files2.sort()

# create the files1
with open("files1.ods", "wb") as final_files1:

    # clean the sample
    for i in files1:
        final_files1.write(open(join(working_dir, i)).read(0x708))


# create the files2
with open("files2.ods", "wb") as final_files2:

    # clean the sample
    for i in files2:
        final_files2.write(open(join(working_dir, i)).read(0x708)) {% endhighlight %}

After running our script we were left with two files:
```
file files1.ods 
files1.ods: OpenDocument Spreadsheet
```
After opening the first file with [Libreoffice](https://fr.libreoffice.org/) we
were greeted by:

<img src="/resources/2016/ndh/catch_me_if_you_can/screen_file1.png" width="800">

Fun isn't it...

Digging in the 2nd file is more profitable, it show us a sort of table with
alphabetic and letter:


<img src="/resources/2016/ndh/catch_me_if_you_can/screen_file2.png" width="800">

if you scroll to the **1048576** line vertical and to the top right most, yes there are
serious... you'll found a "code":
> g6d5g5f2b6g5d3e4d4b3c5b6k2j5j5g4l2 

Using this code with the weird alphbetical table give us the flag: **ndh[wh3re1sw@lly]**.

Challenges resources are available in the [resources
folder](https://github.com/duksctf/duksctf.github.io/tree/master/resources/2016/ndh/catch_me_if_you_can)

