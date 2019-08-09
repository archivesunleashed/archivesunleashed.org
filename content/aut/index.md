---
date: 2016-03-09T19:56:50+01:00
title: The Archives Unleashed Toolkit
weight: 20
---

## Introduction

![Internet Archive server rack](/images/server.jpg)
*Internet Archive servers in San Francisco, photo by Ian Milligan.*

The Archives Unleashed Toolkit is an open-source platform for managing web archives built on [Hadoop](https://hadoop.apache.org/). The platform provides a flexible data model for storing and managing raw content as well as metadata and extracted knowledge. Tight integration with Hadoop provides powerful tools for analytics and data processing via [Spark](http://spark.apache.org/).

Most of this documentation is built on [resilient distributed datasets (RDD)](https://spark.apache.org/docs/latest/rdd-programming-guide.html). We are working on adding support for [DataFrames](http://localhost:1313/aut/spark%20dataframes%20tutorial). You can read more about this in our experimental [DataFrames section](#dataframes).

## Getting Started

### Quick Start

If you don't want to install all the dependencies locally, you can use [`docker-aut`](https://github.com/archivesunleashed/docker-aut). You can run the bleeding edge version of `aut` with `docker run --rm -it archivesunleashed/docker-aut` or a specific version of `aut`, such as 0.17.0 with `docker run --rm -it archivesunleashed/docker-aut:0.17.0`. More information on using `docker-aut`, such as mounting your own data, can be found [here](https://github.com/archivesunleashed/docker-aut#use).

{{< note title="Want a quick walkthrough?" >}}
We have a walkthrough for using AUT on sample data with Docker [here](/aut/lesson).
{{< /note >}}

### Dependencies

The Archives Unleashed Toolkit requires Java.

For Mac OS: You can find information on Java [here](https://java.com/en/download/help/mac_install.xml), or install with [homebrew](https://brew.sh) and then:

```
brew cask install java8
```

For Linux: You can install Java using apt:

```
apt install openjdk-8-jdk
```

Before Spark Shell can launch, JAVA_HOME must be set. If you recieve an error that JAVA_HOME is not set, you need to point it to where Java is installed. On Linux, this might be `export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64` or on Mac OS it might be `export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_74.jdk/Contents/Home`.

### Downloading AUT

The Archives Unleashed Toolkit can be [downloaded as a JAR file for easy use](https://github.com/archivesunleashed/aut/releases/download/aut-0.17.0/aut-0.17.0-fatjar.jar).

The following bash commands will download an example ARC file, and set up a directory to work with AUT. You can also [download the example ARC file here](https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz).

```bash
mkdir aut
cd aut
# example arc file for testing
curl -L "https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz" > example.arc.gz
```

### Installing and Running Spark shell

Remaining in the aut directory you created above, download and unzip [Spark](https://archive.apache.org/dist/spark/spark-2.3.2/spark-2.3.2-bin-hadoop2.7.tgz) from the [Apache Spark Website](http://spark.apache.org/downloads.html).

```bash
curl -L "https://archive.apache.org/dist/spark/spark-2.3.2/spark-2.3.2-bin-hadoop2.7.tgz" > spark-2.3.2-bin-hadoop2.7.tgz
tar -xvf spark-2.3.2-bin-hadoop2.7.tgz
./spark-2.3.2-bin-hadoop2.7/bin/spark-shell --packages "io.archivesunleashed:aut:0.17.0"
```

You should have the spark shell ready and running.

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /___/ .__/\_,_/_/ /_/\_\   version 2.3.2
      /_/

Using Scala version 2.11.8 (Java HotSpot(TM) 64-Bit Server VM, Java 1.8.0_151)
Type in expressions to have them evaluated.
Type :help for more information.

scala>
```

> If you recently upgraded your MacOS, your java version may not be correct in terminal.  You will
> have to [change the path to the latest version in your ./bash_profile file.](https://stackoverflow.com/questions/21964709/how-to-set-or-change-the-default-java-jdk-version-on-os-x).

### Test the Archives Unleashed Toolkit

Type `:paste` at the scala prompt and go into paste mode.

Type or paste the following:

```
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz", sc)
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

### A Note on Memory

As your datasets grow, you may need to provide more memory to Spark shell. You'll know this if you get an error saying that you have run out of "Java Heap Space."

If you're running locally, you can pass it in your startup command like this:

```
./spark-2.3.2-bin-hadoop2.7/bin/spark-shell --driver-memory 4G --packages "io.archivesunleashed:aut:0.17.0"
```

In the above case, you give Spark 4GB of memory to execute the program.

In some other cases, despite giving AUT sufficient memory, you may still encounter Java Heap Space issues. In those cases, it is worth trying to lower the number of worker threads. When running locally (i.e. on a single laptop, desktop, or server), by default AUT runs a number of threads equivalent to the number of cores in your machine.

On a 16-core machine, you may want to drop to 12 cores if you are having memory issues. This will increase stability but decrease performance a bit.

You can do so like this (example is using 12 threads on a 16-core machine):

```
./spark-2.3.2-bin-hadoop2.7/bin/spark-shell --master local[12] --driver-memory 4G --packages "io.archivesunleashed:aut:0.17.0"
```

If you continue to have errors, you may also want to increase the network timeout value. Once in a while, AUT might get stuck on an odd record and take longer than normal to process it. The `--conf spark.network.timeout=10000000` will ensure that AUT continues to work on material, although it may take a while to process. This command then works:

```
./spark-2.3.2-bin-hadoop2.7/bin/spark-shell --master local[12] --driver-memory 90G --conf spark.network.timeout=10000000 --packages "io.archivesunleashed:aut:0.17.0"
```

## Collection Analytics

You may want to get a birds-eye view of your ARCs or WARCs: what top-level domains are included, and at what times were they crawled?

### List of URLs

If you just want a list of URLs in the collection, you can type :p into Spark Shell, paste the script, and then run it with ctrl-d:

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz", sc)
.keepValidPages()
.map(r => r.getUrl)
.take(10)
```

This will give you a list of the top ten URLs. If you want all the URLs, exported to a file, you could run this instead. Note that your export directory cannot already exist.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz", sc)
.keepValidPages()
.map(r => r.getUrl)
.saveAsTextFile("/path/to/export/directory/")
```

### List of Top-Level Domains

You may just want to see the domains within an item. The script below shows the top ten domains within a given file or set of files.

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

If you want to see more than ten results, change the variable in the last line.

### List of Different Subdomains

You can also use regular expressions to extract more fine-tuned information. For example, if you wanted to know all sitenames - i.e. the first-level directories of a given collection.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz", sc)
 .keepValidPages()
 .flatMap(r => """http://[^/]+/[^/]+/""".r.findAllIn(r.getUrl).toList)
 .take(10)
```

In the above example, `"""...."""` declares that we are working with a regular expression, `.r` says turn it into a regular expression, `.findAllIn` says look for all matches in the URL. This will only return the first but that is generally good for our use cases. Finally, `.toList` turns it into a list so you can `flatMap`.

### List of HTTP Status Codes

You may be interested in the [HTTP Status Codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes) of each of the resources. The following script will list the status codes amongst the URLs.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz", sc)
.map(r => (r.getUrl, r.getHttpStatus))
.take(10)
```

### Location of the Resource in ARCs and WARCs

Finally, you may want to know what WARC file the different resources are located in! The following command will provide the full path and filename of the ARC/WARC that each url is found in.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz", sc)
.keepValidPages()
.map(r => (r.getUrl, r.getArchiveFilename))
.take(10)
```

Or, if you just want to know the filename, without the full path and filename, the following script will do that.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._
import org.apache.commons.io.FilenameUtils

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getUrl, FilenameUtils.getName(r.getArchiveFilename)))
  .take(10)
```

## Plain Text Extraction

### All plain text

This script extracts the crawl date, domain, URL, and plain text from HTML files in the sample ARC data (and saves the output to out/). By default, HTTP headers are included in the plain text that is extracted.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(r.getContentString)))
  .saveAsTextFile("plain-text/")
```

If you wanted to use it on your own collection, you would change "src/test/resources/arc/example.arc.gz" to the directory with your own ARC or WARC files, and change "out/" on the last line to where you want to save your output data.

Note that this will create a new directory to store the output, which cannot already exist.

### Plain text without HTTP headers

If you want to remove HTTP headers, you can add one more command: `RemoveHttpHeader`. The script would then look like:

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("plain-text-noheaders/")
```

As most plain text use cases do not require HTTP headers to be in the output, we are removing headers in the following examples.

### Plain text by domain

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a filter string. In the example case, it will go through the collection and find all of the URLs within the "archive.org" domain.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepDomains(Set("www.archive.org"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("plain-text-domain/")
```

### Plain text by URL pattern

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a regular expression pattern. In the example case, it will go through a WARC file and find all of the URLs beginning with `http://archive.org/details/`, and save the text of those URLs.

The `(?i)` makes this query case insensitive.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepUrlPatterns(Set("(?i)http://www.archive.org/details/.*".r))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("details/")
```

### Plain text minus boilerplate

The following Spark script generates plain text renderings for all the web pages in a collection, minus "boilerplate" content: advertisements, navigational elements, and elements of the website template. For more on the boilerplate removal library we are using, [please see this website and paper](http://www.l3s.de/~kohlschuetter/boilerplate/).

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepDomains(Set("www.archive.org"))
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, ExtractBoilerpipeText(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("plain-text-no-boilerplate/")
```

### Plain text filtered by date

AUT permits you to filter records by a list of full or partial date strings. It conceives
of the date string as a `DateComponent`. Use `keepDate` to specify the year (`YYYY`), month (`MM`),
day (`DD`), year and month (`YYYYMM`), or a particular year-month-day (`YYYYMMDD`).

The following Spark script extracts plain text for a given collection by date (in this case, April 2008).

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepDate(List("200804"), ExtractDate.DateComponent.YYYYMM)
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("plain-text-date-filtered-200804/")
```

The following script extracts plain text for a given collection by year (in this case, 2008).

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepDate(List("2008"), ExtractDate.DateComponent.YYYY)
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("plain-text-date-filtered-2008/")
```

Finally, you can also extract multiple dates or years. In this case, we would extract pages from both 2008 and 2015.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepDate(List("2008","2015"), ExtractDate.DateComponent.YYYY)
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
  .saveAsTextFile("plain-text-date-filtered-2008-2015/")
```

Note: if you created just a dump of plain text using another one of the earlier commands, you do not need to go back and run this. You can instead use bash to extract a sample of text. For example, running this command on a dump of all plain text stored in `alberta_education_curriculum.txt`:

```bash
sed -n -e '/^(201204/p' alberta_education_curriculum.txt > alberta_education_curriculum-201204.txt
```

Would select just the lines beginning with `(201204`, or April 2012.

### Plain text filtered by language

The following Spark script keeps only French language pages from a certain top-level domain. It uses the [ISO 639.2 language codes](https://www.loc.gov/standards/iso639-2/php/code_list.php).

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
.keepValidPages()
.keepDomains(Set("www.archive.org"))
.keepLanguages(Set("fr"))
.map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
.saveAsTextFile("plain-text-fr/")
```

### Plain text filtered by keyword

The following Spark script keeps only pages containing a certain keyword, which also stacks on the other scripts.

For example, the following script takes all pages containing the keyword "radio" in a collection.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz",sc)
.keepValidPages()
.keepContent(Set("radio".r))
.map(r => (r.getCrawlDate, r.getDomain, r.getUrl, RemoveHTML(RemoveHttpHeader(r.getContentString))))
.saveAsTextFile("plain-text-radio/")
```

There is also `discardContent` which does the opposite, if you have a frequent keyword you are not interested in.

## Raw HTML Extraction

In most cases, users will be interested in working with plain text. In some cases, however, you may want to work with the acutal HTML of the pages themselves (for example, looking for specific tags or HTML content). 

The following script will produce the raw HTML of a WARC file. You can use the filters from above to filter it down accordingly by domain, language, etc.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, r.getDomain, r.getUrl, r.getContentString))
  .saveAsTextFile("plain-html/")
```

## Named Entity Recognition

{{< warning title="NER is Extremely Resource Intensive and Time Consuming" >}}
Named Entity Recognition is extremely resource intensive, and will take a very long time. Our recommendation is to begin testing NER on one or two WARC files, before trying it on a larger body of information. Depending on the speed of your system, it can take a day or two to process information that you are used to working with in under an hour.
{{< /note >}}


The following Spark scripts use the [Stanford Named Entity Recognizer](http://nlp.stanford.edu/software/CRF-NER.shtml) to extract names of entities – persons, organizations, and locations – from collections of ARC/WARC files or extracted texts. You can find a version of Stanford NER in [our aut-Resources repo located here](https://github.com/archivesunleashed/aut-resources).

The scripts require a NER classifier model. There is one provided in the Stanford NER package (in the `classifiers` folder) called `english.all.3class.distsim.crf.ser.gz`, but you can also use your own.

### Extract entities from ARC/WARC files

```scala
import io.archivesunleashed._
import io.archivesunleashed.app._
import io.archivesunleashed.matchbox._

ExtractEntities.extractFromRecords("/path/to/classifier/english.all.3class.distsim.crf.ser.gz", "example.arc.gz", "output-ner/", sc)
```

Note the call to `addFile()`. This is necessary if you are running this script on a cluster; it puts a copy of the classifier on each worker node. The classifier and input file paths may be local or on the cluster (e.g., `hdfs:///user/joe/collection/`).

The output of this script and the one below will consist of lines that look like this:

```
(20090204,http://greenparty.ca/fr/node/6852?size=display,{"PERSON":["Parti Vert","Paul Maillet","Adam Saab"],
"ORGANIZATION":["GPC Candidate Ottawa Orleans","Contact Cabinet","Accueil Paul Maillet GPC Candidate Ottawa Orleans Original","Circonscriptions Nouvelles Événements Blogues Politiques Contact Mon Compte"],
"LOCATION":["Canada","Canada","Canada","Canada"]})
```

This following script takes the plain text that you may have extracted earlier and extracts the entities.

```scala
import io.archivesunleashed._
import io.archivesunleashed.app._
import io.archivesunleashed.matchbox._

sc.addFile("/path/to/classifier")

ExtractEntities.extractFromScrapeText("english.all.3class.distsim.crf.ser.gz", "/path/to/extracted/text", "output-ner/", sc)
```

## Analysis of Site Link Structure

Site link structures can be very useful, allowing you to learn such things as:

- what websites were the most linked to;
- what websites had the most outbound links;
- what paths could be taken through the network to connect pages;
- what communities existed within the link structure?

Most of the following examples show the **domain** to **domain** links. For example, you discover how many times that `liberal.ca` linked to `twitter.com`, rather than learning that `http://liberal.ca/contact` linked to `http://twitter.com/liberal_party`. The reason we do that is that in general, if you are working with any data at scale, the sheer number of raw URLs can become overwhelming.

We do provide one example below that provides raw data, however.

### Extraction of Simple Site Link Structure

If your web archive does not have a temporal component, the following Spark script will generate the site-level link structure.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._
import io.archivesunleashed.util._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
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
import io.archivesunleashed._
import io.archivesunleashed.matchbox._
import io.archivesunleashed.util._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepContent(Set("apple".r))
  .flatMap(r => ExtractLinks(r.getUrl, r.getContentString))
  .map(r => (ExtractDomain(r._1).removePrefixWWW(), ExtractDomain(r._2).removePrefixWWW()))
  .filter(r => r._1 != "" && r._2 != "")
  .countItems()
  .filter(r => r._2 > 5)

links.saveAsTextFile("links-all-apple/")
```

### Extraction of a Link Structure, using Raw URLs (not domains)

This following script extracts all of the hyperlink relationships between sites, using the full URL pattern.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .flatMap(r => ExtractLinks(r.getUrl, r.getContentString))
  .filter(r => r._1 != "" && r._2 != "")
  .countItems()

links.saveAsTextFile("full-links-all/")
```

You can see that the above was achieved by removing the .map(r => (ExtractDomain(r._1).removePrefixWWW(), ExtractDomain(r._2).removePrefixWWW())) line.

In a larger collection, you might want to add the following line:

```
.filter(r => r._2 > 5)
```

Before `.countItems()` to find just the documents that are linked to more than five times. As you can imagine, raw URLs are very numerous!

### Extraction of a Site Link Structure, organized by URL pattern

In this following example, we run the same script but only extract links coming from URLs matching the pattern `http://www.archive.org/details/*`. We do so by using the `keepUrlPatterns` command.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._
import io.archivesunleashed.util._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepUrlPatterns(Set("(?i)http://www.archive.org/details/.*".r))
  .flatMap(r => ExtractLinks(r.getUrl, r.getContentString))
  .map(r => (ExtractDomain(r._1).removePrefixWWW(), ExtractDomain(r._2).removePrefixWWW()))
  .filter(r => r._1 != "" && r._2 != "")
  .countItems()
  .filter(r => r._2 > 5)

links.saveAsTextFile("details-links-all/")
```

### Grouping by Crawl Date

The following Spark script generates the aggregated site-level link structure, grouped by crawl date (YYYYMMDD). It
makes use of the `ExtractLinks` and `ExtractToLevelDomain` functions.

If you prefer to group by crawl month (YYYMM), replace `getCrawlDate` with `getCrawlMonth` below. If you prefer to group by simply crawl year (YYYY), replace `getCrawlDate` with `getCrawlYear` below.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)
  .saveAsTextFile("sitelinks-by-date/")
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
import io.archivesunleashed._
import io.archivesunleashed.matchbox._
import io.archivesunleashed.matchbox.TupleFormatter._

RecordLoader.loadArchives("/path/to/arc", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)
  .map(tabDelimit(_))
  .saveAsTextFile("sitelinks-tsv/")
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
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .keepUrlPatterns(Set("http://www.archive.org/details/.*".r))
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)
  .saveAsTextFile("sitelinks-details/")
```

### Exporting to Gephi Directly

You may want to export your data directly to the [Gephi software suite](http://gephi.github.io/), an open-soure network analysis project. The following code writes to the GEXF format:

```scala
import io.archivesunleashed._
import io.archivesunleashed.app._
import io.archivesunleashed.matchbox._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .map(r => (r.getCrawlDate, ExtractLinks(r.getUrl, r.getContentString)))
  .flatMap(r => r._2.map(f => (r._1, ExtractDomain(f._1).replaceAll("^\\s*www\\.", ""), ExtractDomain(f._2).replaceAll("^\\s*www\\.", ""))))
  .filter(r => r._2 != "" && r._3 != "")
  .countItems()
  .filter(r => r._2 > 5)

WriteGraph(links, "links-for-gephi.gexf")
```

This file can then be directly opened by Gephi.

We also support exporting to the GraphML format. To do so, the following variation on `WriteGraph` will work:

```scala
WriteGraph.asGraphml(links, "links-for-gephi.graphml")
```

## Image Analysis

AUT supports image analysis, a growing area of interest within web archives.

### Most frequent image URLs in a collection

The following script:

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val links = RecordLoader.loadArchives("example.arc.gz", sc)
  .keepValidPages()
  .flatMap(r => ExtractImageLinks(r.getUrl, r.getContentString))
  .countItems()
  .take(10)
```
Will extract the top ten URLs of images found within a collection, in an array like so:

```
links: Array[(String, Int)] = Array((http://www.archive.org/images/star.png,408), (http://www.archive.org/images/no_star.png,122), (http://www.archive.org/images/logo.jpg,118), (http://www.archive.org/images/main-header.jpg,84), (http://www.archive.org/images/rss.png,20), (http://www.archive.org/images/mail.gif,13), (http://www.archive.org/images/half_star.png,10), (http://www.archive.org/images/arrow.gif,7), (http://ia300142.us.archive.org/3/items/americana/am_libraries.gif?cnt=0,3), (http://ia310121.us.archive.org/2/items/GratefulDead/gratefuldead.gif?cnt=0,3), (http://www.archive.org/images/wayback.gif,2), (http://www.archive.org/images/wayback-election2000.gif,2), (http://www.archive.org/images/wayback-wt...
```

If you wanted to work with the images, you could download them from the Internet Archive.

Let's use the top-ranked example. [This link](http://web.archive.org/web/*/http://archive.org/images/star.png), for example, will show you the temporal distribution of the image. For a snapshot from September 2007, this URL would work:

<http://web.archive.org/web/20070913051458/http://www.archive.org/images/star.png>

To do analysis on all images, you could thus prepend `http://web.archive.org/web/20070913051458/` to each URL and `wget` them en masse.

For more information on `wget`, please consult [this lesson available on the Programming Historian website](http://programminghistorian.org/lessons/automated-downloading-with-wget).

### Most frequent images in a collection, based on MD5 hash

Some images may be the same, but have different URLs. This UDF finds the popular images by calculating the MD5 hash of each and presenting the most frequent images based on that metric. This script:

```scala
import io.archivesunleashed._
import io.archivesunleashed.app._
import io.archivesunleashed.matchbox._

val r = RecordLoader.loadArchives("example.arc.gz",sc).persist()
ExtractPopularImages(r, 500, sc).saveAsTextFile("500-Popular-Images")
```

Will save the 500 most popular URLs to an output directory.

## DataFrames

{{< warning title="Troubleshooting Tips" >}}
This section is experimental and under development! If things don't work, or you have ideas for us, [let us know](https://github.com/archivesunleashed/aut/issues/190)!
{{< /note >}}

There are two main ways to use the Archives Unleashed Toolkit. The above instructions used [resilient distributed datasets (RDD)](https://spark.apache.org/docs/latest/rdd-programming-guide.html).

We are currently developing support for [DataFrames](spark dataframes tutorial). This is still under active development, so syntax may change. We have an [open thread](https://github.com/archivesunleashed/aut/issues/190) in our GitHub repository if you would like to add any suggestions, thoughts, or requests for this functionality.

You will note that right now we do not support everything in DataFrames: we do not support plain text extraction or named entity recognition.

Here we provide some documentation on how to use DataFrames in AUT.

### List of Domains

As with the RDD implementation, the first stop is often to work with the frequency of domains appearing within a web archive. You can see the schema that you can use when working with domains by running the following script:

```scala
import io.archivesunleashed._
import io.archivesunleashed.df._

val df = RecordLoader.loadArchives("example.arc.gz", sc)
  .extractValidPagesDF()

df.printSchema()
```

The below script will show you the top domains within the collection.

```scala
import io.archivesunleashed._
import io.archivesunleashed.df._

val df = RecordLoader.loadArchives("example.arc.gz", sc)
  .extractValidPagesDF()

df.select(ExtractBaseDomain($"Url").as("Domain"))
  .groupBy("Domain").count().orderBy(desc("count")).show()
```

Results will look like:

```
+------------------+-----+
|            Domain|count|
+------------------+-----+
|   www.archive.org|  132|
|     deadlists.com|    2|
|www.hideout.com.br|    1|
+------------------+-----+
```

### Hyperlink Network

You may want to work with DataFrames to extract hyperlink networks. You can see the schema with the following commands: 

```scala
import io.archivesunleashed._
import io.archivesunleashed.df._

val df = RecordLoader.loadArchives("example.arc.gz", sc)
  .extractHyperlinksDF()

df.printSchema()
```

The below script will give you the source and destination for hyperlinks found within the archive.

```scala
import io.archivesunleashed._
import io.archivesunleashed.df._

val df = RecordLoader.loadArchives("example.arc.gz", sc)
  .extractHyperlinksDF()

df.select(RemovePrefixWWW(ExtractBaseDomain($"Src")).as("SrcDomain"),
    RemovePrefixWWW(ExtractBaseDomain($"Dest")).as("DestDomain"))
  .groupBy("SrcDomain", "DestDomain").count().orderBy(desc("SrcDomain")).show()
```

Results will look like:

```
+-------------+--------------------+-----+
|    SrcDomain|          DestDomain|count|
+-------------+--------------------+-----+
|deadlists.com|       deadlists.com|    2|
|deadlists.com|           psilo.com|    2|
|deadlists.com|                    |    2|
|deadlists.com|         archive.org|    2|
|  archive.org|        cyberduck.ch|    1|
|  archive.org|        balnaves.com|    1|
|  archive.org|         avgeeks.com|    1|
|  archive.org|          cygwin.com|    1|
|  archive.org|      onthemedia.org|    1|
|  archive.org|ia311502.us.archi...|    2|
|  archive.org|dvdauthor.sourcef...|    1|
|  archive.org|              nw.com|    1|
|  archive.org|             gnu.org|    1|
|  archive.org|          hornig.net|    2|
|  archive.org|    webreference.com|    1|
|  archive.org|    bookmarklets.com|    2|
|  archive.org|ia340929.us.archi...|    2|
|  archive.org|            mids.org|    1|
|  archive.org|       gutenberg.org|    1|
|  archive.org|ia360602.us.archi...|    2|
+-------------+--------------------+-----+
only showing top 20 rows
```

### Image Analysis

You can also use DataFrames to analyze images. You can see the schema for images by running the following command:

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val df = RecordLoader.loadArchives("example.arc.gz", sc).extractImageDetailsDF();
df.printSchema()
```

The following script will extract all the images, give you their dimensions, as well as unique hashes.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

val df = RecordLoader.loadArchives("example.arc.gz", sc).extractImageDetailsDF();
df.select($"url", $"mime_type", $"width", $"height", $"md5", $"bytes").orderBy(desc("md5")).show()
```

The results will look like this:

```
+--------------------+----------+-----+------+--------------------+--------------------+
|                 url| mime_type|width|height|                 md5|               bytes|
+--------------------+----------+-----+------+--------------------+--------------------+
|http://www.archiv...| image/gif|   21|    21|ff05f9b408519079c...|R0lGODlhFQAVAKUpA...|
|http://www.archiv...|image/jpeg|  275|   300|fbf1aec668101b960...|/9j/4AAQSkZJRgABA...|
|http://www.archiv...|image/jpeg|  300|   225|f611b554b9a44757d...|/9j/4RpBRXhpZgAAT...|
|http://tsunami.ar...|image/jpeg|  384|   229|f02005e29ffb485ca...|/9j/4AAQSkZJRgABA...|
|http://www.archiv...| image/gif|  301|    47|eecc909992272ce0d...|R0lGODlhLQEvAPcAA...|
|http://www.archiv...| image/gif|  140|    37|e7166743861126e51...|R0lGODlhjAAlANUwA...|
|http://www.archiv...| image/png|   14|    12|e1e101f116d9f8251...|iVBORw0KGgoAAAANS...|
|http://www.archiv...|image/jpeg|  300|   116|e1da27028b81db60e...|/9j/4AAQSkZJRgABA...|
|http://www.archiv...|image/jpeg|   84|    72|d39cce8b2f3aaa783...|/9j/4AAQSkZJRgABA...|
|http://www.archiv...| image/gif|   13|    11|c7ee6d7c17045495e...|R0lGODlhDQALALMAA...|
|http://www.archiv...| image/png|   20|    15|c1905fb5f16232525...|iVBORw0KGgoAAAANS...|
|http://www.archiv...| image/gif|   35|    35|c15ec074d95fe7e1e...|R0lGODlhIwAjANUAA...|
|http://www.archiv...| image/png|  320|   240|b148d9544a1a65ae4...|iVBORw0KGgoAAAANS...|
|http://www.archiv...| image/gif|    8|    11|a820ac93e2a000c9d...|R0lGODlhCAALAJECA...|
|http://www.archiv...| image/gif|  385|    30|9f70e6cc21ac55878...|R0lGODlhgQEeALMPA...|
|http://www.archiv...|image/jpeg|  140|   171|9ed163df5065418db...|/9j/4AAQSkZJRgABA...|
|http://www.archiv...|image/jpeg| 1800|    89|9e41e4d6bdd53cd9d...|/9j/4AAQSkZJRgABA...|
|http://www.archiv...| image/gif|  304|    36|9da73cf504be0eb70...|R0lGODlhMAEkAOYAA...|
|http://www.archiv...|image/jpeg|  215|    71|97ebd3441323f9b5d...|/9j/4AAQSkZJRgABA...|
|http://i.creative...| image/png|   88|    31|9772d34b683f8af83...|iVBORw0KGgoAAAANS...|
+--------------------+----------+-----+------+--------------------+--------------------+
only showing top 20 rows
```

You may want to save the images to work with them on your own file system. The following command will save the images from an ARC or WARC.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._
val df = RecordLoader.loadArchives("example.arc.gz", sc).extractImageDetailsDF();
val res = df.select($"bytes").orderBy(desc("bytes")).saveToDisk("bytes", "/path/to/export/directory/")
```

## Loading Data from Amazon S3

We also support loading data stored in [Amazon S3](https://aws.amazon.com/s3/). This advanced functionality requires that you provide spark shell with your AWS Access Key and AWS Secret Key, which you will get when creating your AWS credentials ([read more here](https://aws.amazon.com/blogs/security/wheres-my-secret-access-key/)).

This script, for example, will find the top ten domains from a set of WARCs found in an s3 bucket.

```scala
import io.archivesunleashed._
import io.archivesunleashed.matchbox._

sc.hadoopConfiguration.set("fs.s3a.access.key", "<my-access-key>")
sc.hadoopConfiguration.set("fs.s3a.secret.key", "<my-secret-key>")

val r = RecordLoader.loadArchives("s3a://<my-bucket>/*.gz", sc)
.keepValidPages()
.map(r => ExtractDomain(r.getUrl))
.countItems()
.take(10)
```

You can modify any of the scripts in this documentation accordingly.
