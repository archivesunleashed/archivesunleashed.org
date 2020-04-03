---
title: Getting Started
date: 2019-05-03T09:21:11-05:00
weight: 15
---

<!---
The introductory video will go here
-->

## Overview

The Archives Unleashed Project consists of four major tools: the Archives Unleashed Cloud, the Archives Unleashed Toolkit, the Archives Unleashed Jupyter Notebooks and Warclight.

| Tool                     | Skill        | What it Does | What You Need | Ideal For |
|:--------------------------:|--------------|--------------|---------------|-----------|
| ![logo](/images/cloud-logo.png)  | Beginner     | The **[Archives Unleashed Cloud](/cloud)** is a web-based platform for working with [**Archive-It**](https://archive-it.org) collections. Drawing on your Archive-It credentials, you can sync your collections, run basic analyses, and generate a [standardized set of research derivatives: full text, network diagrams, and basic statistics on your collection](https://cloud.archivesunleashed.org/derivatives).             |  This does not require technical skills. However, you need an **Archive-It** account. You can get this if you are an Archive-It subscriber, **or** if you connect with a librarian responsible for a collection they can generate you a guest account.            | Librarians, and researchers who know a librarian with an Archive-It account!          |
| ![logo](/images/notebook-logo.png) | Beginner/Intermediate | The **[Archives Unleashed Notebooks](/notebooks)** are Jupyter Notebooks that can help you work with the output of the Archives Unleashed Cloud. Once you have them up and running you can use your web browser to work through interactive tutorials! Explore your data through rich visualizations!             |  You need to install the "dependencies" for the notebooks. While you can follow instructions, it does require running commands in your "command line." An intermediate level of technical knowledge is recommended. We [suggest this tutorial](https://programminghistorian.org/en/lessons/intro-to-bash).            | Researchers who want to explore their web archival collections.          |
| ![logo](/images/warclight-logo.png) | Advanced | **[Warclight](/warclight)** is a search engine that lets users discover web archives. Think of it like the library catalogue meeting the WARC file! While it is easy to use, setting it up on your own collections requires an advanced level of knowledge.             | You need a lot of WARCs that would benefit from this search engine. If you don't know what WARCs are, this is not the tool for you!             |  Librarians and archivists who have been collecting web archives and want to enhance collection discoverability.         |
| ![logo](/images/toolkit-logo.png)   | Advanced     |  The **[Archives Unleashed Toolkit](/toolkit)** is an Apache Spark-based platform for analyzing web archives at scale. When you use the Archives Unleashed Cloud, you are using the Toolkit in the back end! As you can see from the documentation page, the Toolkit is very powerful. However, it is an advanced tool that requires a high-level of technical knowledge to use --- or at least, patience and effort. We do have a **hands-on walkthrough** [here](/aut/lesson).            |  You would need WARCs that would benefit from computationally exploring them at scale.            |  Researchers who want to explore their WARCs at scale and need more flexibility than the Cloud provides.         |

## Tools in Action: The Cloud

[Check out the full page here](/cloud).

As noted above, the Archives Unleashed Cloud is an open source cloud-based analysis tool that helps researchers and scholars conduct web archive analysis. **If you have an Archive-It account, you can use it for free at [https://cloud.archivesunleashed.org
](https://cloud.archivesunleashed.org)**.

Here it is in action:

![Cloud collections](/images/collections.png)

![Cloud collections](/images/analysis.png)

![Full graph](/images/neighbours.png)

## Tools in Action: The Notebook

[Check out the full page here](/notebooks).

Archives Unleashed Jupyter Notebooks are a prototype method for working with the derivatives generated by the Archives Unleashed Cloud. They allow you to interactively explore and filter the domain count information, extracted full text, and network visualization data generated by the Cloud.

![AUK Notebook screenshot](/images/AUK_Notebook_Domains.png)

![AUK Notebook screenshot](/images/AUK_Notebook_Text.png)

![AUK Notebook screenshot](/images/AUK_Notebook_Network.png)

## Tools in Action: Warclight

[Check out the full page here](/warclight).

Warclight is an open-source search engine that supports the discovery of web archives held in the WARC and ARC formats. It allows faceted full-text search, record view, and other advanced discovery options.

![Warclight screenshot](/images/warclight.png)

## Tools in Action: The Toolkit

[Check out the full page here](/toolkit).

The Archives Unleashed Toolkit is an open-source toolkit for analyzing web archives built around Apache Spark. 

![Spark Terminal](/images/prompt.png)