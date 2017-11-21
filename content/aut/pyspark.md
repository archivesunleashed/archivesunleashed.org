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

### Installing PySpark

## Collection Analytics

You may want to get a birds-eye view of your ARCs or WARCs: what top-level domains are included, and at what times were they crawled? You can do this in Shell or generate beautiful in-browser visualizations in the Notebook interface.

```python
SCRIPT HERE 
```

### List of URLs 

If you just want a list of URLs in the collection.

```python
SCRIPT HERE 
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