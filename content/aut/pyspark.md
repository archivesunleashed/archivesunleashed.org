---
title: Archives Unleashed Toolkit (PySpark)
date: 2017-11-21T09:22:47-05:00
---

## Introduction

![Internet Archive server rack](/images/server.jpg)
*Internet Archive servers in San Francisco, photo by Ian Milligan.*

The Archives Unleashed Toolkit is an open-source platform for managing web archives built on [Hadoop](https://hadoop.apache.org/). The platform provides a flexible data model for storing and managing raw content as well as metadata and extracted knowledge. Tight integration with Hadoop provides powerful tools for analytics and data processing via [Spark](http://spark.apache.org/).

## Getting Started

### Downloading and Installing AUT

The Archives Unleashed Toolkit, or AUT, can be [downloaded as a JAR file for easy use](https://github.com/archivesunleashed/aut/releases/download/aut-0.10.0/aut-0.10.0-fatjar.jar). 

The use of AUT requires the command line. [This guide](https://programminghistorian.org/lessons/intro-to-bash) provides an introduction to the Bash command line. 

The following bash commands will download the jar and an example ARC file. You can also [download the example ARC file here](https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/arc/example.arc.gz).

```bash
mkdir aut
cd aut
curl -L "https://github.com/archivesunleashed/aut/releases/download/aut-0.10.0/aut-0.10.0-fatjar.jar" > aut-0.10.0-fatjar.jar
#python files in a .zip
curl -L "https://raw.githubusercontent.com/archivesunleashed/aut/master/src/main/python/pyaut.zip" > pyaut.zip
# example arc file for testing
curl -L "https://raw.githubusercontent.com/archivesunleashed/aut/master/src/test/resources/warc/example.warc.gz" > example.warc.gz
```

### Installing the Spark shell

Download and unzip [The Spark Shell](wget http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.6.tgz) from the [Apache Spark Website](http://spark.apache.org/downloads.html).

In the following example, we download Spark Shell and then launch it. Note that we use `pyspark`, which allows you to use the Python programming language to work with Spark. 

It assumes that you are beginning in the `aut` directory that you created above.

```bash
curl -L "http://d3kbcqa49mib13.cloudfront.net/spark-2.1.1-bin-hadoop2.6.tgz" > spark-2.1.1-bin-hadoop2.6.tgz
tar -xvf spark-2.1.1-bin-hadoop2.6.tgz
cd spark-2.1.1-bin-hadoop2.6
./bin/pyspark --jars ../aut-0.10.1-fatjar.jar --driver-class-path ../aut-0.10.1-fatjar.jar --py-files ../pyaut.zip
```
> If for some reason you get `Failed to initialize compiler:
> object scala.runtime in compiler mirror not found.` error,
> this probably means the .jar file did not download properly.
> Try downloading it directly from our [releases page](https://github.com/archivesunleashed/aut/releases/)

If your result looks like this:

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.2.0
      /_/

Using Python version 3.5.2 (default, Jul  2 2016 17:52:12)
SparkSession available as 'spark'.
>>>
```

You should have the spark shell ready and running.


## Testing PySpark

PySpark is slightly different from other Python programs in that it relies on Apache Spark's underlying Scala and Java code to manipulate datasets. You can read more about this in the [Spark documentation](http://spark.apache.org/docs/2.1.0/api/python/pyspark.html).

The other difference between PySpark and the Scala spark-shell is the lack of a way to easily paste code to execute in the shell. 

There are two ways around this.  The easiest way is to create a new python file with your script, and use `spark-submit`. For example, you might create a script with your text editor, save it as `file.py`, and then run it using the following.

```bash
.bin/spark-submit --jars ../aut-0.10.1-fatjar.jar --driver-class-path ../aut-0.10.1-fatjar.jar --py-files ../pyaut.zip /path/to/custom/python/file.py
```

An easier method is the second method: using the interactive, browser-based Jupyter notebooks to work with AUT. 

To get Jupyter running, you will need to install the following. First, you will require a version of Python 3 installed. [You can download it from the Python website](https://www.python.org/downloads/).

For ease of use, you may want to consider using a virtual environment:

```bash
virtualenv python3 ~/.venv_path
. ~/.venv_path/bin/activate
```

This will ensure that you are using Python 3 instead of 2.

You will then need to install the various dependencies that both AUT and Jupyter require. One of the easiest ways to install dependencies for Python is to use pip, a package manager. Luckily, pip is included in the basic installation of Python. If for some reason you do not have it, [this guide] explains how to install it.

Run the following commands:

```bash
pip install Jupyter
pip install numpy
pip install langdetect
pip install bs4
python -m pip install ipykernel
python -m ipykernel install --user
```

With the dependencies downloaded, you are ready to launch your Jupyter notebook. Use the following command, again from your aut directory:

```bash
PYSPARK_DRIVER_PYTHON=jupyter PYSPARK_DRIVER_PYTHON_OPTS=notebook ./bin/pyspark --jars ../aut-0.10.1-fatjar.jar --driver-class-path ../aut-0.10.1-fatjar.jar --py-files ../pyaut.zip
```

Jupyter will start loading at <http://localhost:8888>.  You may be asked for a token upon first launch, which just offers a bit of security. The token is available in the load screen and will look something like this:

```
[C 14:12:08.483 NotebookApp]

  Copy/paste this URL into your browser when you connect for the first time,
  to login with a token:
  http://localhost:8888/?token=cbb021656863dd82a94fe66681b3886fce7af4d230b64abb
```

Near the top right of the Jupyter homepage, you will see "New".

![Notebook screenshot](/images/notebook1.png)

Select a new Python 3 Notebook.  In the top box enter

```python
import RecordLoader
```
and hit Shift-Enter, or press the play button.

If you receive no errors, you are ready to begin working with your web archives!

## Dataframes: Collection Analytics

Pyspark also supports DataFrames, which enable more effective filtering. In this section, we use it to get an overview of what is in a collection. 

For these examples, we are going to use the `example.warc.gz` file that you downloaded above. We assume it is in the `aut` directory, but you can always change the path to it below.

### Setting up the Dataframe

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from pyspark.sql import SparkSession
from pyspark.sql.functions import desc

path = "../example.arc.gz"
spark = SparkSession.builder.appName("extractLinks").getOrCreate()
sc = spark.sparkContext

df = RecordLoader.loadArchivesAsDF (path, sc, spark)
```

### Using DataFrames to Count Domains

Now you can run the following to see the top two domains.

```python
df.groupBy("domain").count().sort(desc("count")).show(n=2)
```

And you'll see:

```
+---------------+-----+
|         domain|count|
+---------------+-----+
|www.archive.org|  132|
|  deadlists.com|    2|
+---------------+-----+
only showing top 2 rows


```

### Using DataFrames to Count Crawl Dates

You can do this for a few other commands as well. For example, by crawl date:

```python
df.groupBy("crawlDate").count().show()
```

And we get:

```
+---------+-----+
|crawlDate|count|
+---------+-----+
| 20080430|  135|
+---------+-----+
```

### Using DataFrames to List URLs

Finally, you can also get a list of the URLs in a collection with the following command:

```
df.select("url").rdd.flatMap(lambda x: x).take(10)
```

You can see a list of the supported Data Frame elements with:

```python
df.printSchema()
```

Which are:

```
root
 |-- contentBytes: binary (nullable = true)
 |-- contentString: string (nullable = true)
 |-- crawlDate: string (nullable = true)
 |-- crawlMonth: string (nullable = true)
 |-- domain: string (nullable = true)
 |-- imageBytes: binary (nullable = true)
 |-- mimeType: string (nullable = true)
 |-- url: string (nullable = true)
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

## Plain Text Extraction

### All plain text

This script extracts the crawl date, domain, URL, and plain text from HTML files in the sample WARC data (and saves the output to a folder called `output-text`).

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from ExtractDate import DateComponent
from RemoveHTML import RemoveHTML
from pyspark.sql import SparkSession

if __name__ == "__main__":
    # replace with your own path to archive file
    path = "../example.arc.gz"
    
    spark = SparkSession.builder.appName("filterByDate").getOrCreate()
    sc = spark.sparkContext

    df = RecordLoader.loadArchivesAsDF(path, sc, spark)
    rdd = df.rdd
    rdd.map(lambda r: (r.crawlDate, r.domain, r.url, RemoveHTML(r.contentString))) \
    .saveAsTextFile("../output-text")
```

### Plain text by domain

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a filter string. In the example case, it will go through the collection and find all of the URLs within the "deadlists.com" domain (which can be found in the example WARC file).

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from ExtractDate import DateComponent
from RemoveHTML import RemoveHTML
from pyspark.sql import SparkSession

if __name__ == "__main__":
    # replace with your own path to archive file
    path = "../example.arc.gz"
    
    spark = SparkSession.builder.appName("filterByDate").getOrCreate()
    sc = spark.sparkContext

    df = RecordLoader.loadArchivesAsDF(path, sc, spark)
    rdd = df.filter(df["domain"].like("%deadlist%")).rdd
    rdd.map(lambda r: (r.crawlDate, r.domain, r.url, RemoveHTML(r.contentString))) \
    .saveAsTextFile("../output-text-deadlist")
```

### Plain text by URL pattern

The following Spark script generates plain text renderings for all the web pages in a collection with a URL matching a regular expression pattern. In the example case, it will go through the collection and find all of the URLs containing "http://www.archive.org/iathreads". 

```python
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from ExtractDate import DateComponent
from RemoveHTML import RemoveHTML
from pyspark.sql import SparkSession

if __name__ == "__main__":
    # replace with your own path to archive file
    path = "../example.arc.gz"
    
    spark = SparkSession.builder.appName("filterByDate").getOrCreate()
    sc = spark.sparkContext

    df = RecordLoader.loadArchivesAsDF(path, sc, spark)
    rdd = df.filter(df["url"].like("%http://www.archive.org/iathreads%")).rdd
    rdd.map(lambda r: (r.crawlDate, r.domain, r.url, RemoveHTML(r.contentString))) \
    .saveAsTextFile("../output-text-threads")
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
import RecordLoader
from DFTransformations import *
from ExtractDomain import ExtractDomain
from ExtractLinks import ExtractLinks
from ExtractDate import DateComponent
from RemoveHTML import RemoveHTML
from pyspark.sql import SparkSession

if __name__ == "__main__":
    # replace with your own path to archive file
    path = "../example.arc.gz"
    
    spark = SparkSession.builder.appName("filterByDate").getOrCreate()
    sc = spark.sparkContext

    df = RecordLoader.loadArchivesAsDF(path, sc, spark)
    filtered_df = keepDate(df, "2008", DateComponent.YYYY)
    rdd = filtered_df.rdd
    rdd.map(lambda r: (r.crawlDate, r.domain, r.url, RemoveHTML(r.contentString))) \
    .saveAsTextFile("../output-text")
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

If your web archive does not have a temporal component, the following Spark script will generate the site-level link structure.

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
