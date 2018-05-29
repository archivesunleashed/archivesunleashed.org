---
title: Hands on With The Archives Unleashed Toolkit
date: 2018-05-29T11:51:41-04:00
---

Welcome to the Archives Unleashed Toolkit hands-on walkthrough!

![Spark Terminal](/images/prompt.png)
The reality of any hands-on workshop is that things will break. We've tried our best to provide a robust environment that can let you walk through the basics of the Archives Unleashed Toolkit alongside us.

If you have any questions, let [Nick Ruest](https://github.com/ruebot) or [Ian Milligan](https://github.com/ianmilligan1) know.

## Table of Contents
- [Installation and Use](#installation)
- [Hello World: Our First Script](#hello-world)
- [Extracting Some Text](#extracting-text)
	- [Ouch! Our First Error](#error)
	- [Other Text Analyis Filters](#other-filters)
- [People, Places, and Things! Entities Ahoy!](#entities)
- [Web of Links: Network Analysis](#network)
- [Acknowlegements and Final Notes](#conclusion)

## Installation and Use {#installation}

This lesson only requires one dependency: [Docker](https://www.docker.com/get-docker). 

Later in this lesson, we use the networking tool [Gephi](https://gephi.org/).

Make sure that Docker is running! If it isn't you might see an error like `docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?` – make sure to run it (on Mac, for example, you need to run the Docker application itself).

Make a directory in your userspace, somewhere where you can find it: on your desktop, perhaps. Call it `data`. In my case, I will create it on my desktop and it will have a path like `/Users/ianmilligan1/desktop/data`.

Use the following command, replacing `/path/to/your/data` with the directory. **If you want to use your own ARC or WARC files, please put them in this directory**.

`docker run --rm -it -v "/path/to/your/data:/data" archivesunleashed/docker-aut`

For example, if your files are in `/Users/ianmilligan1/desktop/data` you would run the above command like:

`docker run --rm -it -v "/Users/ianmilligan1/desktop/data:/data" archivesunleashed/docker-aut`

Once you run this command, you will have to wait a few minutes while data is downloaded and AUT builds. Once it is all working, you should see:

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.1.1
      /_/

Using Scala version 2.11.8 (OpenJDK 64-Bit Server VM, Java 1.8.0_151)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

## Hello World: Our First Script {#hello-world}

Now that we are at the prompt, let's get used to running commands. The easiest way to use the Spark Shell is to _copy and paste_ scripts that you've written somewhere else in.

Fortunately, the Spark Shell supports this functionality!

At the `scala>` prompt, type the following command and press enter.

```
:paste
```

Now cut and paste the following script:

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._

val r = RecordLoader.loadArchives("/aut-resources/Sample-Data/*.gz", sc)
.keepValidPages()
.map(r => ExtractDomain(r.getUrl))
.countItems()
.take(10)
```

Let's take a moment to look at this script. It:

* begins by importing the AUT libraries;
* tells the program where it can find the data (in this case, the sample data that we have included in this Docker image);
* tells it only to keep the "valid" pages, in this case HTML data;
* tells it to `ExtractDomain`, or find the base domain of each URL - i.e. `www.google.com/cats` we are interested just in the domain, or `www.google.com`;
* count them - how many times does `www.google.com` appear in this collection, for example;
* and display the top ten!

Once it is pasted in, let's run it.

You run pasted scripts by holding the *Ctrl* key and the *D* key at the same time. Try that now.

You should see:

```
// Exiting paste mode, now interpreting.

import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._
r: Array[(String, Int)] = Array((www.equalvoice.ca,4644), (www.liberal.ca,1968), (greenparty.ca,732), (www.policyalternatives.ca,601), (www.fairvote.ca,465), (www.ndp.ca,417), (www.davidsuzuki.org,396), (www.canadiancrc.com,90), (www.gca.ca,40), (communist-party.ca,39))

scala>
```

We like to use this example to do two things:

* It is fairly simple and lets us know that AUT is working;
* and it tells us what we can expect to find in the web archives! In this case, we have a lot of the Liberal Party of Canada, Equal Voice Canada, and the Green Party of Canada.

**If you loaded your own data above**, you can access that directory by substituting the directory in the `loadArchives` command. Try it again! Remeber to type `:paste`, paste the following command in, and then `ctrl` + `D` to execute.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._

val r = RecordLoader.loadArchives("/data/*.gz", sc)
.keepValidPages()
.map(r => ExtractDomain(r.getUrl))
.countItems()
.take(10)
```

## Extracting some Text {#extracting-text}

Now that we know what we might find in a web archive, let us try extracting some text. You might want to get just the text of a given website or domain, for example.

Above we learned that the Liberal Party of Canada's website has 1,968 captures in the sample files we provided. Let's try to just extract that text.

To load this script, remember: type `paste`, copy-and-paste it into the shell, and then hold `ctrl` and `D` at the same time.

```scala
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("/aut-resources/Sample-Data/*.gz", sc)
  .keepValidPages()
  .keepDomains(Set("liberal.ca"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("/data/liberal-party-text")
```

**If you're using your own data, that's why the domain count was key!** Swap out the "liberal.ca" command above with the domain that you want to look at from your own data.

Now let's look at the ensuing data. Go to the folder you provided in the very first startup – remember, in my case it was `/users/ianmilligan1/desktop/data` - and you will now have a folder called `liberal-party-text`. Open up the files with your text editor and check it out!

### Ouch: Our First Error {#error}

One of the vexing parts of this interface is that it creates output directories – and if the directory already exists, it comes tumbling down.

As this is one of the most common errors, let's see it and then learn how to get around it.

Try running the **exact same script** that you did above.

```scala
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("/aut-resources/Sample-Data/*.gz", sc)
  .keepValidPages()
  .keepDomains(Set("liberal.ca"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("/data/liberal-party-text")
```

Instead of a nice crisp feeling of success, you will see a long dump of text beginning with:

```
org.apache.hadoop.mapred.FileAlreadyExistsException: Output directory file:/data/liberal-party-text already exists
```

To get around this, you can do two things:

* Delete the existing directory that you created;
* Change the name of the output file - to `/data/liberal-party-text-2` for example.

Good luck!

### Other Text Analysis Filters {#other-filters}

Take some time to explore the various options and variables that you can swap in and around the `.keepDomains` line. Check out the [documentation](http://archivesunleashed.org/aut/#plain-text-extraction) for some ideas.

Some options:

* **Keep URL Patterns**: Instead of domains, what if you wanted to have text relating to just a certain pattern? Substitute `.keepDomains` for a command like: `.keepUrlPatterns(Set("(?i)http://geocities.com/EnchantedForest/.*".r))`
* **Filter by Date**: What if we just wanted data from 2005, or 2008? You could add the following command after `.keepValidPages()`: `.keepDate(List("2005"), YYYY)`
* **Filter by Language**: What if you just want French-language pages? After `.keepDomains` add a new line: `.keepLanguages(Set("fr"))`.

For example, if we just wanted the French-language Liberal pages, we would run:

```scala
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("/aut-resources/Sample-Data/*.gz", sc)
  .keepValidPages()
  .keepDomains(Set("liberal.ca"))
  .keepLanguages(Set("fr"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("/data/liberal-party-french-text")
```

## People, Places, and Things: Entities Ahoy! {#entities}

One last thing we can do with text is to try to use [Named-entity recognition](https://en.wikipedia.org/wiki/Named-entity_recognition) (NER) to try to find people, organizations, and locations within the text.

To do this, we need to have a classifier - luckily, we have included an English-language one from the [Stanford NER project](https://nlp.stanford.edu/software/CRF-NER.shtml) in this Docker image!

The code is below. It looks a bit different than what you are used to:

```
import io.archivesunleashed.spark.matchbox.ExtractEntities

ExtractEntities.extractFromRecords("/aut-resources/NER/english.all.3class.distsim.crf.ser.gz", "/aut-resources/Sample-Data/*.gz", "/data/ner-output/", sc)
```

This will take a fair amount of time, even on a small amount of data. It is very computationally intensive! I often use it as an excuse to go make a cup of coffee.

When it is done, in the /data file you will have results. The first line should look like:

```
(20060622,http://www.gca.ca/indexcms/?organizations&orgid=27,{"PERSON":["Marie"],"ORGANIZATION":["Green Communities Canada","Green Communities Canada News and Events Our Programs Join Green Communities Canada Downloads Privacy Policy Site Map GCA Clean North Kathie Brosemer"],"LOCATION":["St. E. Sault","Canada"]})
```

Here we can see that in this website, it was probably taking about Sault Ste. Marie, Ontario. 

## Web of Links: Network Analysis {#network}

One other thing we can do is a network analysis. By now you are probably getting good at running code.

Let's extract all of the links from the sample data and export them to a file format that the popular network analysis program Gephi can use.

```scala
import io.archivesunleashed.spark.matchbox.{ExtractDomain, ExtractLinks, RecordLoader, WriteGEXF}
import io.archivesunleashed.spark.rdd.RecordRDD._

val links = RecordLoader.loadArchives("/aut-resources/Sample-Data/*.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)

WriteGEXF(links, "/data/links-for-gephi.gexf")
```

By now this should be seeming pretty straightforward! (remember to keep using `:paste` to enter this code).

Now let's use these instructions to [work with Gephi](https://ianmilligan.ca/2015/12/11/from-dataverse-to-gephi-network-analysis-on-our-data/).

## Acknowledgements and Final Notes {#conclusion}

The ARC and WARC file are drawn from the [Canadian Political Parties & Political Interest Groups Archive-It Collection](https://archive-it.org/collections/227), collected by the University of Toronto. We are grateful that they've provided this material to us.

If you use their material, please cite it along the following lines:

- University of Toronto Libraries, Canadian Political Parties and Interest Groups, Archive-It Collection 227, Canadian Action Party, http://wayback.archive-it.org/227/20051004191340/http://canadianactionparty.ca/Default2.asp

You can find more information about this collection at [WebArchives.ca](http://webarchives.ca/). 

This work is primarily supported by the [Andrew W. Mellon Foundation](https://mellon.org/). Other financial and in-kind support comes from the [Social Sciences and Humanities Research Council](http://www.sshrc-crsh.gc.ca), [Compute Canada](https://www.computecanada.ca), the [Ontario Ministry of Research, Innovation, and Science](https://www.ontario.ca/page/ministry-research-innovation-and-science), [York University Libraries](https://www.library.yorku.ca/web/), [Smart Start Labs](http://www.startsmartlabs.com), and the [Faculty of Arts](https://uwaterloo.ca/arts/) and [David R. Cheriton School of Computer Science](https://cs.uwaterloo.ca) at the [University of Waterloo](https://uwaterloo.ca).