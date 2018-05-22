---
date: 2016-03-09T19:56:50+01:00
title: The Archives Unleashed Cloud
weight: 20
---

![Cloud portal](/images/portal.png)

## Introduction

Imagine a world where you could access and analyze a web archives file in as few as three clicks, with no coding necessary. Oh wait, there's a portal for that: the Archives Unleashed Cloud, or AUK.

AUK is an open source cloud-based analysis tool that helps researchers and scholars conduct web archive analysis. It supports the priorities of accessibility and usability of web archives by providing users a web-based front end to access the [Archives Unleashed Toolkit](/aut). It has been primarily developed by our project co-investigator and developer, [Nick Ruest](http://ruebot.net/).

TL;DR? You can use the Cloud at <http://cloud.archivesunleashed.org>

## Functionalities

AUK has a clean and modern interface with an easy-to-use dashboard. The best part is that you do not need to know how to code, which in tern supports the increased accessibility of working with web archives.

Core features of the Archives Unleashed Cloud include:

* Syncing Archive-It collections;
* Ingesting Archive-It collections via WASAPI to AUK;
* Creating a network graph, domain distribution list, full-text derivatives, and making each available for download;
* Providing an in-browser network diagram to see major nodes and connections within your collection.

In addition, AUK's [documentation](http://cloud.archivesunleashed.org/documentation) offers guidance on how to get AUK up and running, as well as how to use some of its features. 

## A Guided Tour of AUK

Let's take a tour of how AUK works. Once a user signs up to [cloud.archivesunleashed.org](http://cloud.archivesunleashed.org/), they enter their Archive-It credentials, which are salted and encrypted. Those credentials are then used to sync their Archive-It collections with AUK using Archive-It's [WASAPI](https://github.com/WASAPI-Community/data-transfer-apis) endpoint. This is done as a background job, and once it is complete, it emails the user to let them know that their Archive-It collections are synced and available for further analysis on their dashboard. 

The AUK dashboard provides some basic information about each collection: title, if it is publicly available (in Archive-It), the number of ARC/WARCs in the collection, and the size of the collection. You can see this below!

![Cloud collections](/images/collections.png)

From the dashboard, users can select a collection to download, which will trigger a number of background jobs. The first job uses data gathered from the WASAPI endpoint to download and verify each ARC/WARC file to our AUK instance. Once the entire collection is downloaded, an automatic email is generated to notify the user that the collection has been downloaded, and analysis will begin. 

The analysis process then triggers an Apache Spark job and uses AUT to create a basic set of derivatives:

* A GEXF file which you can load with Gephi. It has a basic layout courtesy of our [Graphpass](https://github.com/archivesunleashed/graphpass) program, which allows you to see major nodes and communities in the network;
* A GraphML file which you can load with [Gephi](https://gephi.org/). It does not have any basic layouts or transformations, requiring you to do so manually. You can use Graphpass to provide layout if you wish to add that feature to your file;
* A csv file that explains the distribution of domains within the web archive;
* A txt file that contains the plain text extracted from HTML documents within the web archive. You can find the crawl date, full URL, and the plain text of each page within the file.

Here is a completed collection page:

![Cloud collections](/images/analysis.png)

In addition, we use [Graphpass](https://github.com/archivesunleashed/graphpass) to help to create a simple network visualization powered by [Sigma js](http://sigmajs.org/) on the collection dashboard, and display a table of the top 10 domains occurrences in a given collection.

![Full graph](/images/graph.png)

Future development will focus on filtering further down on a collection, and integrating the new [DataFrame](https://spark.apache.org/docs/latest/sql-programming-guide.html) functionality we're adding to AUT via a [JDBC](https://en.wikipedia.org/wiki/JDBC_driver) connector.

## Can I try it out?

Currently, anyone with an Archive-It subscription can take AUK for a spin. Please just come over to <http://cloud.archivesunleashed.org> and give it a try.

If you don't have an Archive-It account, sit tight, we are looking into development for AUK users who would like to bring their own WARCs to AUK with [webrecorder.io](https://webrecorder.io/). Updates will be provided through our social media channels and newsletter, as well as on our website.

AUK is an open source project, you can view the codebase [here](http://github.com/archivesunleashed/auk). Although it is tied closely to the canonical instance running at <http://cloud.archivesunleashed.org>, it can also be run as a standalone project on your own server, desktop, or laptop! That said, our primary focus is on the canonical instance that we are hosting. However, if there is interest in generalizing aspects of the project, [let us know and we can collaborate](https://archivesunleashed.org/get-involved/) and figure out how to make it happen.
