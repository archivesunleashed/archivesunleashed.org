---
date: 2016-03-09T19:56:50+01:00
title: The Archives Unleashed Toolkit
weight: 20
---

## Introduction

![Internet Archive server rack](/images/server.jpg)
*Internet Archive servers in San Francisco, photo by Ian Milligan.*

The Archives Unleashed Toolkit is an open-source platform for managing web archives built on [Hadoop](https://hadoop.apache.org/). The platform provides a flexible data model for storing and managing raw content as well as metadata and extracted knowledge. Tight integration with Hadoop provides powerful tools for analytics and data processing via [Spark](http://spark.apache.org/).

## Getting Started

### Downloading AUT

The Archives Unleashed Toolkit can be [downloaded as a JAR file for easy use](https://github.com/archivesunleashed/aut/releases/download/aut-0.11.0/aut-0.11.0-fatjar.jar). 

The following bash commands will download the jar and an example ARC file. You can also [download the example ARC file here](https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz).

```bash
mkdir aut
cd aut
curl -L "https://github.com/archivesunleashed/aut/releases/download/aut-0.11.0/aut-0.11.0-fatjar.jar" > aut-0.11.0-fatjar.jar
# example arc file for testing
curl -L "https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz" > example.arc.gz
```

### Installing Spark shell

Download and unzip [The Spark Shell](wget http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.6.tgz) from the [Apache Spark Website](http://spark.apache.org/downloads.html).

```bash
curl -L "http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.6.tgz" > spark-2.1.1-bin-hadoop2.6.tgz
tar -xvf spark-2.1.1-bin-hadoop2.6.tgz
cd spark-2.1.1-bin-hadoop2.6
./bin/spark-shell --jars ../aut-0.11.0-fatjar.jar
```
> If for some reason you get `Failed to initialize compiler: 
> object scala.runtime in compiler mirror not found.` error, 
> this probably means the .jar file did not download properly.
> Try downloading it directly from our [releases page](https://github.com/archivesunleashed/aut/releases/)

You should have the spark shell ready and running.

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.1.1
      /_/

Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_151)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

> If you recently upgraded your Mac OS X, your java version may not be correct in terminal.  You will 
> have to [change the path to the latest version in your ./bash_profile file.](https://stackoverflow.com/questions/21964709/how-to-set-or-change-the-default-java-jdk-version-on-os-x).

### Test the Archives Unleashed Toolkit

Type `:paste` at the scala prompt and go into paste mode.

Type or paste the following:

```
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._

val r = RecordLoader.loadArchives("../example.arc.gz", sc)
.keepValidPages()
.map(r => ExtractDomain(r.getUrl))
.countItems()
.take(10)
```

then `<ctrl> d` to exit paste mode and run the script.

If you see:

```
r: Array[(String, Int)] = Array((www.archive.org,132), (deadlists.com,2), (www.hideout.com.br,1))

```

That means you're up and running!

You should now be able to try out the toolkit's many tutorials. 

## Collection Analytics

You may want to get a birds-eye view of your ARCs or WARCs: what top-level domains are included, and at what times were they crawled? 

### List of URLs 

If you just want a list of URLs in the collection, you can type :p into Spark Shell, paste the script, and then run it with ctrl-d:

```scala
import io.archivesunleashed.spark.matchbox._ 
import io.archivesunleashed.spark.rdd.RecordRDD._ 

val r = RecordLoader.loadArchives("../example.arc.gz", sc) 
.keepValidPages()
.map(r => r.getUrl)
.take(10)
```

This will give you a list of the top ten URLs. If you want all the URLs, exported to a file, you could run this instead. Note that your export directory cannot already exist.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._

val r = RecordLoader.loadArchives("../example.arc.gz", sc) 
.keepValidPages()
.map(r => r.getUrl)
.saveAsTextFile("/path/to/export/directory/")
```

### List of Top-Level Domains

You may just want to see the domains within an item. The script below shows the top ten domains within a given file or set of files.

```scala
import io.archivesunleashed.spark.matchbox._ 
import io.archivesunleashed.spark.rdd.RecordRDD._ 

val r = 
RecordLoader.loadArchives("../example.arc.gz", sc) 
.keepValidPages() 
.map(r => ExtractDomain(r.getUrl)) 
.countItems() 
.take(10) 
```

If you want to see more than ten results, change the variable in the last line. 

### List of Different Subdomains

Finally, you can use regular expressions to extract more fine-tuned information. For example, if you wanted to know all sitenames - i.e. the first-level directories of a given collection.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._

val r = RecordLoader.loadArchives("../example.arc.gz", sc)
 .keepValidPages()
 .flatMap(r => """http://[^/]+/[^/]+/""".r.findAllIn(r.getUrl).toList)
```

In the above example, `"""...."""` declares that we are working with a regular expression, `.r` says turn it into a regular expression, `.findAllIn` says look for all matches in the URL. This will only return the first but that is generally good for our use cases. Finally, `.toList` turns it into a list so you can `flatMap`.

## Plain Text Extraction

### All plain text

This script extracts the crawl date, domain, URL, and plain text from HTML files in the sample ARC data (and saves the output to out/). 

```scala
import io.archivesunleashed.spark.rdd.RecordRDD._
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}

RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("out/")
```

If you wanted to use it on your own collection, you would change "src/test/resources/arc/example.arc.gz" to the directory with your own ARC or WARC files, and change "out/" on the last line to where you want to save your output data.

Note that this will create a new directory to store the output, which cannot already exist.

### Plain text by domain

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a filter string. In the example case, it will go through the collection and find all of the URLs within the "archive.org" domain.

```scala
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .keepDomains(Set("archive.org"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("out/")
```

### Plain text by URL pattern

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a regular expression pattern. In the example case, it will go through the collection and find all of the URLs beginning with `http://geocities.com/EnchantedForest/`. The `(?i)` makes this query case insensitive.

```scala
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("/path/to/many/warcs/*.gz", sc)
  .keepValidPages()
  .keepUrlPatterns(Set("(?i)http://geocities.com/EnchantedForest/.*".r))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("EnchantedForest/")
```

### Plain text minus boilerplate

The following Spark script generates plain text renderings for all the web pages in a collection, minus "boilerplate" content: advertisements, navigational elements, and elements of the website template. For more on the boilerplate removal library we are using, [please see this website and paper](http://www.l3s.de/~kohlschuetter/boilerplate/).

```scala
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader, ExtractBoilerpipeText}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .keepDomains(Set("archive.org"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, ExtractBoilerpipeText(r.getContentString)))
  .saveAsTextFile("out/")
```

### Plain text filtered by date

AUT permits you to filter records by a full or partial date string. It conceives
of the date string as a `DateComponent`. Use `keepDate` to specify the year (`YYYY`), month (`MM`),
day (`DD`), year and month (`YYYYMM`), or a particular year-month-day (`YYYYMMDD`).

The following Spark script extracts plain text for a given collection by date (in this case, 4 October 2008). 

```scala
import io.archivesunleashed.spark.matchbox.RecordLoader
import io.archivesunleashed.spark.rdd.RecordRDD._
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.matchbox.ExtractDate.DateComponent._

RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .keepDate("20081004", YYYYMM)
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("out/")
```

The following script extracts plain text for a given collection by year (in this case, 2016).

```scala
import io.archivesunleashed.spark.matchbox.RecordLoader
import io.archivesunleashed.spark.rdd.RecordRDD._
import io.archivesunleashed.spark.matchbox.{RemoveHTML, RecordLoader}
import io.archivesunleashed.spark.matchbox.ExtractDate.DateComponent._

RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .keepDate("2015", YYYY)
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("out2/")
```

Note: if you created just a dump of plain text using another one of the earlier commands, you do not need to go back and run this. You can instead use bash to extract a sample of text. For example, running this command on a dump of all plain text stored in `alberta_education_curriculum.txt`:

```bash
sed -n -e '/^(201204/p' alberta_education_curriculum.txt > alberta_education_curriculum-201204.txt
```

Would select just the lines beginning with `(201204`, or April 2012.

### Plain text filtered by language

The following Spark script keeps only French language pages from a certain top-level domain. It uses the [ISO 639.2 language codes](https://www.loc.gov/standards/iso639-2/php/code_list.php).

```scala
import io.archivesunleashed.spark.matchbox.{RecordLoader, RemoveHTML}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("../example.arc.gz", sc)
.keepValidPages()
.keepDomains(Set("archive.org"))
.keepLanguages(Set("fr"))
.map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
.saveAsTextFile("out-fr/")
```

### Plain text filtered by keyword

The following Spark script keeps only pages containing a certain keyword, which also stacks on the other scripts. 

For example, the following script takes all pages containing the keyword "archive" in a collection.

```scala
import io.archivesunleashed.spark.matchbox._ 
import io.archivesunleashed.spark.rdd.RecordRDD._ 

val r = RecordLoader.loadArchives("../example.arc.gz",sc)
.keepValidPages()
.keepContent(Set("archive".r))
.map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
.saveAsTextFile("out-archive/")
```

There is also `discardContent` which does the opposite, if you have a frequent keyword you are not interested in.

## Named Entity Recognition

The following Spark scripts use the [Stanford Named Entity Recognizer](http://nlp.stanford.edu/software/CRF-NER.shtml) to extract names of entities – persons, organizations, and locations – from collections of ARC/WARC files or extracted texts. You can find a version of Stanford NER in [our aut-Resources repo located here](https://github.com/archivesunleashed/aut-resources).

The scripts require a NER classifier model. There is one provided in the Stanford NER package (in the `classifiers` folder) called `english.all.3class.distsim.crf.ser.gz`, but you can also use your own.

## Extract entities from ARC/WARC files

```scala
import io.archivesunleashed.spark.matchbox.ExtractEntities

sc.addFile("/path/to/classifier")

ExtractEntities.extractFromRecords("english.all.3class.distsim.crf.ser.gz", "/path/to/arc/or/warc/files", "output/", sc)
```

Note the call to `addFile()`. This is necessary if you are running this script on a cluster; it puts a copy of the classifier on each worker node. The classifier and input file paths may be local or on the cluster (e.g., `hdfs:///user/joe/collection/`).

The output of this script and the one below will consist of lines that look like this:

```
(20090204,http://greenparty.ca/fr/node/6852?size=display,{"PERSON":["Parti Vert","Paul Maillet","Adam Saab"],
"ORGANIZATION":["GPC Candidate Ottawa Orleans","Contact Cabinet","Accueil Paul Maillet GPC Candidate Ottawa Orleans Original","Circonscriptions Nouvelles Événements Blogues Politiques Contact Mon Compte"],
"LOCATION":["Canada","Canada","Canada","Canada"]})
```

```scala
import io.archivesunleashed.spark.matchbox.ExtractEntities

sc.addFile("/path/to/classifier")

ExtractEntities.extractFromScrapeText("english.all.3class.distsim.crf.ser.gz", "/path/to/extracted/text", "output/", sc)
```

## Analysis of Site Link Structure

Site link structures can be very useful, allowing you to learn such things as:

- what websites were the most linked to;  
- what websites had the most outbound links;  
- what paths could be taken through the network to connect pages;  
- what communities existed within the link structure?  

### Extraction of Simple Site Link Structure

If your web archive does not have a temporal component, the following Spark script will generate the site-level link structure. In this case, it is loading all WARCs stored in an example collection of GeoCities files.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._
import StringUtils._

val links = RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .flatMap(r => ExtractLinks(r.getUrl, r.getContentString))
  .map(r => (ExtractDomain(r._1).removePrefixWWW(), ExtractDomain(r._2).removePrefixWWW()))
  .filter(r => r._1 != "" && r._2 != "")
  .countItems()
  .filter(r => r._2 > 5)

links.saveAsTextFile("links-all/")
```

Note how you can add filters. In this case, we add a filter so you are looking at a network graph of pages containing the phrase "apple." Filters can go immediately after `.keepValidPages()`.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._
import StringUtils._

val links = RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .keepContent(Set("apple".r))
  .flatMap(r => ExtractLinks(r.getUrl, r.getContentString))
  .map(r => (ExtractDomain(r._1).removePrefixWWW(), ExtractDomain(r._2).removePrefixWWW()))
  .filter(r => r._1 != "" && r._2 != "")
  .countItems()
  .filter(r => r._2 > 5)

links.saveAsTextFile("links-all/")
```

### Extraction of a Site Link Structure, organized by URL pattern

In this following example, we run the same script but only extract links coming from URLs matching the pattern `http://geocities.com/EnchantedForest/.*`. We do so by using the `keepUrlPatterns` command.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._
import StringUtils._

val links = RecordLoader.loadArchives("/path/to/many/warcs/*.gz", sc)
  .keepValidPages()
  .keepUrlPatterns(Set("http://geocities.com/EnchantedForest/.*".r))
  .flatMap(r => ExtractLinks(r.getUrl, r.getContentString))
  .map(r => (ExtractDomain(r._1).removePrefixWWW(), ExtractDomain(r._2).removePrefixWWW()))
  .filter(r => r._1 != "" && r._2 != "")
  .countItems()
  .filter(r => r._2 > 5)

links.saveAsTextFile("geocities-links-all/")
```

### Grouping by Crawl Date

The following Spark script generates the aggregated site-level link structure, grouped by crawl date (YYYYMMDD). It
makes use of the `ExtractLinks` and `ExtractToLevelDomain` functions.

If you prefer to group by crawl month (YYYMM), replace `getCrawlDate` with `getCrawlMonth` below. If you prefer to group by simply crawl year (YYYY), replace `getCrawlDate` with `getCrawlYear` below.

```scala
import io.archivesunleashed.spark.matchbox.{ExtractDomain, ExtractLinks, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._

RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)
  .saveAsTextFile("sitelinks")
```

The format of this output is:
- Field one: Crawldate, yyyyMMdd
- Field two: Source domain (i.e. liberal.ca)
- Field three: Target domain of link (i.e. ndp.ca)
- Field four: number of links.

```
((20080612,liberal.ca,liberal.ca),1832983)
((20060326,ndp.ca,ndp.ca),1801775)
((20060426,ndp.ca,ndp.ca),1771993)
((20060325,policyalternatives.ca,policyalternatives.ca),1735154)
```

In the above example, you are seeing links within the same domain.

Note also that `ExtractLinks` takes an optional third parameter of a base URL. If you set this – typically to the source URL –
ExtractLinks will resolve a relative path to its absolute location. For example, if
`val url = "http://mysite.com/some/dirs/here/index.html"` and `val html = "... <a href='../contact/'>Contact</a> ..."`, and we call `ExtractLinks(url, html, url)`, the list it returns will include the 
item `(http://mysite.com/a/b/c/index.html, http://mysite.com/a/b/contact/, Contact)`. It may
be useful to have this absolute URL if you intend to call `ExtractDomain` on the link
and wish it to be counted.


### Exporting as TSV
Archive records are represented in Spark as [tuples](https://en.wikipedia.org/wiki/Tuple), 
and this is the standard format of results produced by most of the scripts presented here
(e.g., see above). It may be useful, however, to have this data in TSV (tab-separated value)
format, for further processing outside AUT. The following script uses `tabDelimit` (from
`TupleFormatter`) to transform tuples to tab-delimited strings; it also flattens any 
nested tuples. (This is the same script as at the top of the page, with the addition of the 
third and the second-last lines.)

```scala
import io.archivesunleashed.spark.matchbox.{ExtractDomain, ExtractLinks, RecordLoader}
import io.archivesunleashed.spark.rdd.RecordRDD._
import io.archivesunleashed.spark.matchbox.TupleFormatter._

RecordLoader.loadArchives("/path/to/arc", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)
  .map(tabDelimit(_))
  .saveAsTextFile("cpp.sitelinks2")
```

Its output looks like:
```
20151107        liberal.ca      youtube.com     16334
20151108        socialist.ca    youtube.com     11690
20151108        socialist.ca    ustream.tv      11584
20151107        canadians.org   canadians.org   11426
20151108        canadians.org   canadians.org   11403
```

### Filtering by URL
In this case, you would only receive links coming from websites in matching the URL pattern listed under `keepUrlPatterns`.

```scala
import io.archivesunleashed.spark.matchbox.{ExtractDomain, ExtractLinks, RecordLoader, WriteGDF}
import io.archivesunleashed.spark.rdd.RecordRDD._

val links = RecordLoader.loadArchives("/path/to/many/warcs/*.gz", sc)
  .keepValidPages()
  .keepUrlPatterns(Set("http://liberal.ca/Canada/.*".r))
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)
  .saveAsTextFile("cpp.sitelinks-liberal")
```

### Exporting to Gephi Directly

You may want to export your data directly to the [Gephi software suite](http://gephi.github.io/), an open-soure network analysis project. The following code writes to the GDF format:

```scala
import io.archivesunleashed.spark.matchbox.{ExtractDomain, ExtractLinks, RecordLoader, WriteGDF}
import io.archivesunleashed.spark.rdd.RecordRDD._

val links = RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)

WriteGDF(links, "links-for-gephi.gdf")
```

This file can then be directly opened by Gephi.

We also support exporting to the GEXF file format, which will be available in the next release (or you can build the project yourself to access it, or just e-mail [mailto:i2millig@uwaterloo.ca](Ian Milligan) for help). 

## Image Analysis

Warcbase supports image analysis, a growing area of interest within web archives.  

### Most frequent image URLs in a collection

The following script:

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._

val links = RecordLoader.loadArchives("../example.arc.gz", sc)
  .keepValidPages()
  .flatMap(r => ExtractImageLinks(r.getUrl, r.getContentString))
  .countItems()
  .take(10) 
```
Will extract the top ten URLs of images found within a collection, in an array like so:

>links: Array[(String, Int)] = Array((http://www.archive.org/images/star.png,408), (http://www.archive.org/images/no_star.png,122), (http://www.archive.org/images/logo.jpg,118), (http://www.archive.org/images/main-header.jpg,84), (http://www.archive.org/images/rss.png,20), (http://www.archive.org/images/mail.gif,13), (http://www.archive.org/images/half_star.png,10), (http://www.archive.org/images/arrow.gif,7), (http://ia300142.us.archive.org/3/items/americana/am_libraries.gif?cnt=0,3), (http://ia310121.us.archive.org/2/items/GratefulDead/gratefuldead.gif?cnt=0,3), (http://www.archive.org/images/wayback.gif,2), (http://www.archive.org/images/wayback-election2000.gif,2), (http://www.archive.org/images/wayback-wt...

If you wanted to work with the images, you could download them from the Internet Archive. 

Let's use the top-ranked example. [This link](http://web.archive.org/web/*/http://archive.org/images/star.png), for example, will show you the temporal distribution of the image. For a snapshot from September 2007, this URL would work:

<http://web.archive.org/web/20070913051458/http://www.archive.org/images/star.png>

To do analysis on all images, you could thus prepend `http://web.archive.org/web/20070913051458/` to each URL and `wget` them en masse.

For more information on `wget`, please consult [this lesson available on the Programming Historian website](http://programminghistorian.org/lessons/automated-downloading-with-wget). 

### Most frequent images in a collection, based on MD5 hash

Some images may be the same, but have different URLs. This UDF finds the popular images by calculating the MD5 hash of each and presenting the most frequent images based on that metric. This script:

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.rdd.RecordRDD._
import io.archivesunleashed.spark.matchbox.RecordLoader

val r = RecordLoader.loadArchives("../example.arc.gz",sc).persist()
ExtractPopularImages(r, 500, sc).saveAsTextFile("500-Popular-Images")
```

Will save the 500 most popular URLs to an output directory.

## Twitter Analysis

AUT also supports parsing and analysis of large volumes of Twitter JSON. This allows you to work with social media and web archiving together on one platform. We are currently in active development. If you have any suggestions or want more features, feel free to pitch in at [our AUT repository](https://github.com/archivesunleashed/aut).

### Gathering Twitter JSON Data

To gather Twitter JSON, you will need to use the Twitter API to gather information. We recommend [twarc](https://github.com/edsu/twarc), a "command line tool (and Python library) for archiving Twitter JSON." Nick Ruest and Ian Milligan wrote an open-access article on using twarc to archive an ongoing event, which [you can read here](https://github.com/web-archive-group/ELXN42-Article/blob/master/elxn42.md). 

For example, with twarc, you could begin using the searching API (stretching back somewhere between six and nine days) on the #elxn42 hashtag with:

```
twarc.py --search "#elxn42" > elxn42-search.json
```

Or you could use the streaming API with:

```
twarc.py --stream "#elxn42" > elxn42-stream.json
```

Functionality is similar to other parts of AUT, but note that you use `loadTweets` rather than `loadArchives`. 

### Basic Twitter Analysis

With the ensuing JSON file (or directory of JSON files), you can use the following scripts. Here we're using the "top ten", but you can always save all of the results to a text file if you desire.

### An Example script, annotated

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.matchbox.TweetUtils._
import io.archivesunleashed.spark.rdd.RecordRDD._

// Load tweets from HDFS
val tweets = RecordLoader.loadTweets("/path/to/tweets", sc)

// Count them
tweets.count()

// Extract some fields
val r = tweets.map(tweet => (tweet.id, tweet.createdAt, tweet.username, tweet.text, tweet.lang,
                             tweet.isVerifiedUser, tweet.followerCount, tweet.friendCount))

// Take a sample of 10 on console
r.take(10)

// Count the different number of languages
val s = tweets.map(tweet => tweet.lang).countItems().collect()

// Count the number of hashtags
// (Note we don't 'collect' here because it's too much data to bring into the shell)
val hashtags = tweets.map(tweet => tweet.text)
                     .filter(text => text != null)
                     .flatMap(text => {"""#[^ ]+""".r.findAllIn(text).toList})
                     .countItems()

// Take the top 10 hashtags
hashtags.take(10)
```

The above script does the following:

* loads the tweets; 
* counts them; 
* extracts specific fields based on the Twitter JSON; 
* Samples them; 
* counts languages; 
* and counts and lets you know the top 10 hashtags in a collection. 

### Parsing a Specific Field

For example, a user may want to parse a specific field. Here we explore the `created_at` field. 

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.matchbox.TweetUtils._
import io.archivesunleashed.spark.rdd.RecordRDD._
import java.text.SimpleDateFormat
import java.util.TimeZone

val tweets = RecordLoader.loadTweets("/shared/uwaterloo/uroc2017/tweets-2016-11", sc)

val counts = tweets.map(tweet => tweet.createdAt)
  .mapPartitions(iter => {
      TimeZone.setDefault(TimeZone.getTimeZone("UTC"))
      val dateIn = new SimpleDateFormat("EEE MMM dd HH:mm:ss ZZZZZ yyyy")
      val dateOut = new SimpleDateFormat("yyyy-MM-dd")
    iter.map(d => try { dateOut.format(dateIn.parse(d)) } catch { case e: Exception => null })})
  .filter(d => d != null)
  .countItems()
  .sortByKey()
  .collect()
```

The next example takes the parsed `created_at` field with some of the earlier elements to see how often the user @HillaryClinton (or any other user) was mentioned in a corpus.

```scala
import io.archivesunleashed.spark.matchbox._
import io.archivesunleashed.spark.matchbox.TweetUtils._
import io.archivesunleashed.spark.rdd.RecordRDD._
import java.text.SimpleDateFormat
import java.util.TimeZone

val tweets = RecordLoader.loadTweets("/shared/uwaterloo/uroc2017/tweets-2016-11/", sc)

val clintonCounts = tweets
  .filter(tweet => tweet.text != null && tweet.text.contains("@HillaryClinton"))
  .map(tweet => tweet.createdAt)
  .mapPartitions(iter => {
      TimeZone.setDefault(TimeZone.getTimeZone("UTC"))
      val dateIn = new SimpleDateFormat("EEE MMM dd HH:mm:ss ZZZZZ yyyy")
      val dateOut = new SimpleDateFormat("yyyy-MM-dd")
    iter.map(d => try { dateOut.format(dateIn.parse(d)) } catch { case e: Exception => null })})
  .filter(d => d != null)
  .countItems()
  .sortByKey()
  .collect()
```

### Parsing JSON

What if you want to do more and access more data inside tweets?
Tweets are just JSON objects, see examples
[here](https://gist.github.com/hrp/900964) and
[here](https://gist.github.com/gnip/764239).  Twitter has [detailed
API documentation](https://dev.twitter.com/overview/api/tweets) that
tells you what all the fields mean.

The Archives Unleashed Toolkit internally uses
[json4s](https://github.com/json4s/json4s) to access fields in
JSON. You can manipulate fields directly to access any part of tweets.
Here are some examples:

```scala
import org.json4s._
import org.json4s.jackson.JsonMethods._

val sampleTweet = """  [insert tweet in JSON format here] """
val json = parse(sampleTweet)
```

The you can do something like:

```scala
implicit lazy val formats = org.json4s.DefaultFormats

// Extract id
(json \ "id_str").extract[String]

// Extract created_at
(json \ "created_at").extract[String]
```
