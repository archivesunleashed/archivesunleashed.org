---
title: Archives Unleashed Toolkit (PySpark)
date: 2017-11-21T09:22:47-05:00
---

## Introduction

![Internet Archive server rack](/images/server.jpg)
*Internet Archive servers in San Francisco, photo by Ian Milligan.*

The Archives Unleashed Toolkit is an open-source platform for managing web archives built on [Hadoop](https://hadoop.apache.org/). The platform provides a flexible data model for storing and managing raw content as well as metadata and extracted knowledge. Tight integration with Hadoop provides powerful tools for analytics and data processing via [Spark](http://spark.apache.org/).

## Getting Started

### Downloading AUT

PySpark support of aut requires:
* The Archives Unleashed Toolkit release >= 0.10.1

The Archives Unleashed Toolkit can be [downloaded as a JAR file for easy use](https://github.com/archivesunleashed/aut/releases/download/aut-0.10.0/aut-0.10.0-fatjar.jar).

### Installing PySpark

The following bash commands will download the jar and an example ARC file. You can also [download the example ARC file here](https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz).

```bash
mkdir aut
cd aut
curl -L "https://github.com/archivesunleashed/aut/releases/download/aut-0.10.0/aut-0.10.0-fatjar.jar" > aut-0.10.0-fatjar.jar
#python files in a .zip
curl -L "https://raw.githubusercontent.com/archivesunleashed/aut/master/src/main/python/pyaut.zip" > pyaut.zip
# example arc file for testing
curl -L "https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz" > example.arc.gz
```

### Installing Spark shell

Download and unzip [The Spark Shell](wget http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.6.tgz) from the [Apache Spark Website](http://spark.apache.org/downloads.html).

```bash
curl -L "http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.6.tgz" > spark-2.1.1-bin-hadoop2.6.tgz
tar -xvf spark-2.1.1-bin-hadoop2.6.tgz
cd spark-2.1.1-bin-hadoop2.6
./bin/pyspark --jars ../aut-0.10.1-fatjar.jar --driver-class ../aut-0.10.1-fatjar.jar --py-files ../pyaut.zip
```
> If for some reason you get `Failed to initialize compiler:
> object scala.runtime in compiler mirror not found.` error,
> this probably means the .jar file did not download properly.
> Try downloading it directly from our [releases page](https://github.com/archivesunleashed/aut/releases/)

If your result looks like this:

```
```

You should have the spark shell ready and running.


## Testing PySpark

PySpark is slightly different from other Python programs in that it relies on Apache Spark's underlying Scala and Java code to manipulate datasets. You can read more about this in the Spark documentation (http://spark.apache.org/docs/2.1.0/api/python/pyspark.html).  

The other difference between PySpark and the Scala spark-shell is the lack of a :paste feature in the shell. There are two ways around this.  The easiest way is to create a new python file with your script, and use `spark-submit`.

```bash
.bin/spark-submit --jars ../aut-0.10.1-fatjar.jar --driver-class ../aut-0.10.1-fatjar.jar --py-files ../pyaut.zip /path/to/custom/python/file.py
```

This tutorial will apply the second method -- using Jupyter notebooks / IPython.
To get Jupyter running, you will need to install the following in Python3:

First it is a good idea to start a virtual environment

```bash
virtualenv python3 ~/.venv_path
. ~/.venv_path/bin/activate
```
This will ensure that you are using Python 3 instead of 2.  Then install the dependencies:

```bash
pip install Jupyter
pip install numpy
pip install langdetect
pip install bs4
```
Then finally setup ipykernel and launch jupyter.

```bash
python -m pip install ipykernel
python -m ipykernel install --user
PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS=notebook ./bin/pyspark --jars ../aut-0.10.1-fatjar.jar --driver-class ../aut-0.10.1-fatjar.jar --py-files ../pyaut.zip
```

Jupyter will start loading at http://localhost:8888.  You may be asked for a token upon first launch.  The token is available in the load screen and will look something like this:

```bash
[C 14:12:08.483 NotebookApp]

  Copy/paste this URL into your browser when you connect for the first time,
  to login with a token:
  http://localhost:8888/?token=cbb021656863dd82a94fe66681b3886fce7af4d230b64abb
```

Near the top right of the Jupyter homepage, you will see "New".

(image here)

Select a new Python 3 Notebook.  In the top box enter

```python
import RecordLoader
```
and hit Shift-Enter.

If you receive no errors, you are ready to start the tutorial!

## Collection Analytics

Similar to the Scala script, you can get a basic overview of what is in your Warc or Arc files.  The following script will give you the most frequent links between one website and an other, count them and show the top 10.

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from pyspark.sql import SparkSession

path = "../example.arc.gz"
spark = SparkSession.builder.appName("extractLinks").getOrCreate()
sc = spark.sparkContext
rdd = RecordLoader.loadArchivesAsRDD(path, sc, spark)\
      .flatMap(lambda r: ExtractLinks(r.url, r.contentString))\
      .map(lambda r: (ExtractDomain(r[0]), ExtractDomain(r[1])))\
      .filter(lambda r: r[0] is not None and r[0]!= "" and r[1] is not None and r[1] != "")

print(countItems(rdd).filter(lambda r: r[1] > 5).take(10))
```

### Dataframe support

Pyspark also supports dataframes, which enable more effective filtering.

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from pyspark.sql import SparkSession

path = "../example.arc.gz"
spark = SparkSession.builder.appName("extractLinks").getOrCreate()
sc = spark.sparkContext

df = RecordLoader.loadArchivesAsDF (path, sc, spark)
df.select(df['url'], df['contentString'])
```

### Turn Your WARCs into Temporary Database Table

```python

import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from pyspark.sql import SparkSession

path = "../example.arc.gz"
spark = SparkSession.builder.appName("extractLinks").getOrCreate()
sc = spark.sparkContext
df = RecordLoader.loadArchivesAsDF (path, sc, spark)

df.createOrReplaceTempView("warc") # create a table called "warc"
dfSQL = spark.sql('SELECT * FROM warc WHERE domain="www.archive.org" GROUP BY crawlMonth')
dfSQL.show()
```


### List of URLs

If you just want a list of URLs in the collection.  The following script will give a list of tuples with a url name and the count for each occurrence.

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from pyspark.sql import SparkSession
from RemoveHTML import RemoveHTML

rdd = RecordLoader.loadArchivesAsRDD(path, sc, spark)\
        .flatMap(lambda r: ExtractLinks(r.url, r.contentString))\
        .flatMap(lambda r: (ExtractDomain(r[0]), ExtractDomain(r[1])))\
        .filter(lambda r: r is not None and r != "")
print(countItems(rdd).take(100))

```

### List of Different Subdomains

If you want a list of domains in the collection.

```python
SCRIPT HERE
```

## Plain Text Extraction

### All plain text

This script extracts the crawl date, domain, URL, and plain text from HTML files in the sample ARC data (and saves the output to out/).

```python
SCRIPT HERE
```

### Plain text by domain

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a filter string. In the example case, it will go through the collection and find all of the URLs within the "greenparty.ca" domain.

```python
SCRIPT HERE
```

### Plain text by URL pattern

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a regular expression pattern. In the example case, it will go through the collection and find all of the URLs beginning with `http://geocities.com/EnchantedForest/`. The `(?i)` makes this query case insensitive.

```python
SCRIPT HERE
```

### Plain text minus boilerplate

The following Spark script generates plain text renderings for all the web pages in a collection, minus "boilerplate" content: advertisements, navigational elements, and elements of the website template. For more on the boilerplate removal library we are using, [please see this website and paper](http://www.l3s.de/~kohlschuetter/boilerplate/).

```python
SCRIPT HERE
```

### Plain text filtered by date

AUT permits you to filter records by a full or partial date string. It conceives
of the date string as a `DateComponent`. Use `keepDate` to specify the year (`YYYY`), month (`MM`),
day (`DD`), year and month (`YYYYMM`), or a particular year-month-day (`YYYYMMDD`).

The following script extracts plain text for a given collection by date (in this case, 4 October 2008).

```python
SCRIPT HERE
```

### Plain text filtered by language

The following Spark script keeps only French language pages from a certain top-level domain. It uses the [ISO 639.2 language codes](https://www.loc.gov/standards/iso639-2/php/code_list.php).

```python
SCRIPT HERE
```

### Plain text filtered by keyword

The following Spark script keeps only pages containing a certain keyword, which also stacks on the other scripts.

For example, the following script takes all pages containing the keyword "guestbooks" in a collection.

```python
SCRIPT HERE
```

There is also `discardContent` which does the opposite, if you have a frequent keyword you are not interested in.

## Named Entity Recognition

The following Spark scripts use the [Stanford Named Entity Recognizer](http://nlp.stanford.edu/software/CRF-NER.shtml) to extract names of entities – persons, organizations, and locations – from collections of ARC/WARC files or extracted texts. You can find a version of Stanford NER in [our aut-Resources repo located here](https://github.com/archivesunleashed/aut-resources).

The scripts require a NER classifier model. There is one provided in the Stanford NER package (in the `classifiers` folder) called `english.all.3class.distsim.crf.ser.gz`, but you can also use your own.

```python
SCRIPT HERE
```

### Extract entities from ARC/WARC files

## Analysis of Site Link Structure

Site link structures can be very useful, allowing you to learn such things as:

- what websites were the most linked to;  
- what websites had the most outbound links;  
- what paths could be taken through the network to connect pages;  
- what communities existed within the link structure?  

### Extraction of Simple Site Link Structure

If your web archive does not have a temporal component, the following Spark script will generate the site-level link structure. In this case, it is loading all WARCs stored in an example collection of GeoCities files.

```python
SCRIPT here
```

Note how you can add filters. In this case, we add a filter so you are looking at a network graph of pages containing the phrase "apple." Filters can go immediately after `.keepValidPages()`.

### Extraction of a Site Link Structure, organized by URL pattern

In this following example, we run the same script but only extract links coming from URLs matching the pattern `http://geocities.com/EnchantedForest/.*`. We do so by using the `keepUrlPatterns` command.

```python
SCRIPT HERE
```

### Grouping by Crawl Date

The following Spark script generates the aggregated site-level link structure, grouped by crawl date (YYYYMMDD). It
makes use of the `ExtractLinks` and `ExtractToLevelDomain` functions.

If you prefer to group by crawl month (YYYMM), replace `getCrawlDate` with `getCrawlMonth` below. If you prefer to group by simply crawl year (YYYY), replace `getCrawlDate` with `getCrawlYear` below.

```python
SCRIPT HERE
```

### Filtering by URL
In this case, you would only receive links coming from websites in matching the URL pattern listed under `keepUrlPatterns`.

```python
SCRIPT HERE
```

## Image Analysis

Warcbase supports image analysis, a growing area of interest within web archives.  

```python
SCRIPT HERE
```

### Most frequent image URLs in a collection

The following script will extract the top ten URLs of images found within a collection.

```python
SCRIPT HERE
```

## Troubleshooting

### Import error: RecordLoader not found

If you attempted to compress the pyaut folder on your own,
