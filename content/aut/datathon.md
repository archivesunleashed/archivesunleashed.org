---
title: Archives Unleashed Datathon Cheat Sheet
date: 2019-01-01T11:51:41-04:00
---

This guide is meant to be a quick reference guide for attendees at the Archives Unleashed [datathons](/events). These are some commonly-asked questions and options that attendees have used.

Screenshots are from Mac, as we don't have a Windows machine on our project team. Our apologies.

## Finding Your Terminal

While the [Archives Unleashed Cloud](/cloud) is easy to use in a web browser, the [Archives Unleashed Toolkit](/toolkit) requires the use of the command line.

On Mac OS, you can find the terminal by going to your "Applications" folder, then "Utilities," then "Terminal." See the screenshot below.

![Download](/images/Mac-OS-terminal.png)

On Windows, we recommend using PowerShell. You can launch this a few different ways, as outlined in this [guide from Microsoft](https://docs.microsoft.com/en-us/powershell/scripting/getting-started/starting-windows-powershell?view=powershell-6). We recommend clicking your "Start" button, typing "PowerShell", and then clicking "Windows PowerShell".

## Connecting to a Virtual Machine

Fortunately, we do most of our work at the datathon not on your own laptop, but on our [Compute Canada](https://www.computecanada.ca) virtual machines! 

The trick is connecting to them. 

To do so, you need two things:

* a file known as a private key so that you can connect to the machine securely (at our hackathons, this is generally known as `archives-hackathon.key`)
* an IP address that lets your computer know _where_ the machine you are joining is.

We will give you the key at the datathon. We will also assign each team a virtual machine known by its IP. When you receive the key, we recommend saving it to your desktop for convenience during the event.

## Setting Up the Key

The first thing you need to do with your key is change the ownership. You need to open your terminal window and type the following. This will work on your desktop.

```bash
chmod 600 ~/desktop/archives-hackathon.key
```

It might look a bit like this. No error message means that it worked!

![Download](/images/chmod.png)

## Connecting

To connect, you need to modify the following command. Note that the IP address will change. 

```bash
ssh -i ~/desktop/archives-hackathon.key ubuntu@206.167.180.200
```

The IP address is the number `206.167.180.200` above. If this works, you'll see something like this.

```
The authenticity of host '18.236.128.116 (18.236.128.116)' can't be established.
ECDSA key fingerprint is SHA256:YjkquLPPmNlFqU4+E4cscSA9lqbHqg3D/8pBN0F+sRg.
Are you sure you want to continue connecting (yes/no)? yes
```

Type yes.

In the future, logging in will now look like this.

![Download](/images/login.png)

## Launching the Toolkit on the Virtual Machine

Once you're on the machine, it will be preloaded with a number of collections - we will let you know the details about that in Slack.

Each of the VMs will be preloaded with:

- Apache Spark 2.3.2
  - Spark shell: `/home/ubuntu/spark/bin/spark-shell`
- Python3, Python 2.7
- Java 8
- Ruby 2.3.1
- jq
- GNU Parallel

Data will be located in `/mnt/data` in a series of subdirectories. For example, a collection might be found in `/mnt/data/sfu-ngos` for the SFU NGO collection.

A few other things to keep in mind:

- Each collection's directory will have a `warcs` directory containing all of its ARCs/WARCs
- Each collection's directory will have a `derivatives` directory containing the scholarly derivatives created on <https://cloud.archivesunleashed.org>.
- More info about those files can be found here: <https://cloud.archivesunleashed.org/derivatives>.

To run Apache Spark shell with `aut` on each machine, run the following command:
`~/spark/bin/spark-shell --packages "io.archivesunleashed:aut:0.17.0"`

In the course of your project, you might need to use additional flags. These should work well on each machine:
`~/spark/bin/spark-shell --packages "io.archivesunleashed:aut:0.17.0" --master local[*] --driver-memory 12G --conf spark.network.timeout=10000000`

The `--master local[*]` refers to the number of cores that you are using. In general, the default works. But if you run into crashes, you may want to lower that number to `6`, i.e. `--master local[6]`.

The `--driver-memory 12G` refers to the amount of RAM you've given the toolkit. On our VMs, please keep it at `12G`. 

## What to do with AUT?

We recommend using the [main documentation found here](https://archivesunleashed.org/aut/). Take a look at the left hand column and see what you can do with WARCs. 

Let's take a sample script, however. In the documentation [here](https://archivesunleashed.org/aut/#collection-analytics), see this script that lets you know what domains are found.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r =
RecordLoader.loadArchives("example.arc.gz", sc)
.keepValidPages()
.map(r => ExtractDomain(r.getUrl))
.countItems()
.take(10)
```

To run this on your machine, you just need to change the input. So let's change `example.arc.gz` to the directory that has your data. In this case, we'll use the fictional SFU NGO collection mentioned above.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r =
RecordLoader.loadArchives("/mnt/data/sfu-ngos/warcs/*.gz", sc)
.keepValidPages()
.map(r => ExtractDomain(r.getUrl))
.countItems()
.take(10)
```

## Copying Results From the VM

Finally, you may want to get results out of the VM so that you can use them locally.

Let's imagine you have a result file on your VM named `results.txt`. In this case, it is located in a directory called `output` off your home.

Here I `cd`, or change directory, into that directory and then type `pwd` to get the full path of it.

![Download](/images/pwd.png)

I now know that the file I want is located as:

```
/home/ubuntu/output/results.txt
```

To get that file on my own machine, I open up a **NEW** terminal window. In this case, I decide I want to move it to my downloads folder. 

I first type

```
cd downloads
```

to move to my downloads folder. I then run `scp` to transfer the file. Note that it looks _very similar_ to the command you used to connect to the file.

```bash
scp -i ~/desktop/archives-hackathon.key ubuntu@18.236.128.116:/home/ubuntu/output/results.txt .
```

The trailing `.` is not a typo. This tells it to download in the directory you are in! See the screenshot below.

![Download](/images/download.png)