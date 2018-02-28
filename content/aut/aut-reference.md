# Archives Unleashed Library

AUT is a collection of tools for filtering, aggregating, analyzing and
visualizing web archive data. The primary structure is contained in three
folders -- The Archive Record Class, Matchbox, RDD Tools, and Utilities.

## The Archive Record Class

`import io.archivesunleashed.spark.archive.io.ArchiveRecord`

`class ArchiveRecord(r: SerializableWritable[ArchiveRecordWritable]) extends Serializable`

### Description

The `ArchiveRecord` class is used by `RecordLoader` to manage the ingestion of
WARC and ARC files.  It represents one record from a collection found in an .arc
or .warc file. It uses the ARCRecord & WARCRecord apis to extract data from
webcrawls such as the crawl date, content strings, urls and so on.

`ArchiveRecord` is a generic record that covers data from both ARCRecord and
WARCRecord format.  AUT will convert either format into the generic `ArchiveRecord`
class.

### Variables

- `.arcRecord: ARCRecord` : if one is detected, stores a record from an Arc File.
- `.warcRecord: WARCRecord` : if one is detected, stores a record from a Warc File.
- `.ISO8602: SimpleDateFormat` : Format for CrawlDates. \"yyyy\-MM\-dd\'T\'HH:mm:ssX\" by default.
- `.getCrawlDate: String` : Extract the CrawlDate from the record with format YYYYMMDD.
- `.getCrawlMonth: String` : Extract the CrawlDate from the record with format YYYYMM.
- `.getContentBytes: Array[Byte]` : Extract the (html) content as a byte array.
- `.getContentString: String` : Extract the (html) content as a string.
- `.getMimeType: String` : Extract the MimeType (e.g. "text/html" or "video/mp4").
- `.getUrl : String` : Extract the source url (where the crawl occurred).
- `.getDomain : String` : Extract the source domain (see ExtractDomain).
- `.getImageBytes : Array[Byte]` : Extract Image data as a byte array.

## The Archives Unleashed Matchbox

`import io.archivesunleashed.spark.matchbox`

### Description

The matchbox contains a collection of tools to manipulate data that comes from
one or more ArchiveRecords.  The most important tool is the `RecordLoader` which takes a
file or files in a path and returns a RDD of `ArchiveRecord`.

### Loading ArchiveRecords

- `RecordLoader` : The RecordLoader is usually the first call when using aut. It loads a WARC or ARC file and puts it into a spark RDD for quick iterating and derivative creation.
    * `loadArchives(path: String, sc: SparkContext, keepValidPages: Boolean = true): RDD[ArchiveRecord]` : load a WARC or ARC into Spark. By default, it will remove any invalid pages from the analysis, unless keepValidPages is false.
    * `loadTweets(path: String, sc: SparkContext): RDD[JValue]` : load JSON from a file that contains Twitter Api data and store it in an RDD.  Uses `json4s` and `json4s.jackson.JsonMethods`.

### Text Utilities

- `DetectLanguage(input:String): String` : Detects the language of a text input. Returns the two-character language code for the detected language.
- `DetectMimeTypeTika(content: String): String` : Detects the Mime Type using Tika.
- `ExtractBoilerpipeText(String)` : Extracts boilerplate text (e.g. a generic copyright statement)
from website content.
- `ExtractAtMentions(String): List[String]` : Extracts all `@username` from a Twitter status update or other content string, returning a list of users.
- `ExtractDate(fullDate :String, dateFormat: DateComponent): String` : Extract a date from a string in the desired format (`YYYY`, `MM`, `DD`, `YYYYMM`, or `YYYYMMDD`).
- `ExtractDomain(url: String, source: String=""): String` : Extracts the domain host (www.example.com)
from a full url (www.example.com/additional/file/path/info).  If desired, an additional string
can be passed to the function as a default for the case the url input returns `null`.
- `ExtractEntities
    * (iNerClassifierFile: String, inputRecordFile: String, outputFile: String, sc: SparkContext)` : Conduct [Named Entity Recogition](https://nlp.stanford.edu/software/CRF-NER.html) classification on a WARC or ARC record using the path to a [NER classifier](https://stanfordnlp.github.io/CoreNLP/), an ARC or WARC file path, an output file and a spark context.  
- `ExtractGraph(records: RDD[ArchiveRecord], dynamic: Boolean = false, tolerance : Double = 0.005, numIter: Int = 20): Graph[VertexData, EdgeData]` : Extract a social network graph based on the Crawled collections and any links found in the html. Optionally, can calculate page ranks if `dynamic` is set to `true`. In this case, the tolerance and number of iterations for the page rank calculation can be controlled using `tolerance` and `numIter`. Calculating page rank is not recommended for a very large number of ArchiveRecords as it can take a long time to calculate.
    * `.writeAsJson(verticesPath: String, edgesPath: String)` : Takes an extracted graph and saves it into two files, one for vertices and another for edges.
- `ExtractHashtags(src: String): List[String]` : Extract all `#hashtags` from a Twitter status update or other content string, returning a list of hashtags.
- `ExtractLinks(src: String): List[String]` : Extract all urls from a content string, returning a list of detected urls.
- `RemoveHTML (content : String)` : Removes html tags from content data.
- `RemoveHttpHeader ( content: String): String` : Removes the http header (http:// or https://) from a url string.
- `StringUtils` : Tools for modifying strings.
    * `.removePrefixWWW() : String` : Removes the "www." from a url if it contains one.  This avoids duplication of websites (www.example.com vs example.com).
    * `.escapeInvalidXML() : String` : Converts invalid XML tags \&, \", \<, \> to XML-safe entities (&amp;, &quot;, &lt;, &gt;). May be able to resolve some issues due to corrupt data.
    * `.computeHash() : String` : Converts a string to an MD5 checksum.  Used to create ids in GEXF or Graphml outputs, but can also be used to anonymize data.
- `TweetUtils` : Tools for manipulating Twitter API Json Data.
    * `.id()` : extract the Twitter id.
    * `.createdAt()` : get the data a tweet was posted.
    * `.text()` : get the text from the tweet.
    * `.lang` : get the language.
    * `.username()` : get the username of the person who posted the tweet.
    * `.isVerifiedUser` : `true` if the user is a "Verified User" otherwise, `false`.
    * `.followerCount` : the number of followers for the Twitter user.
    * `.friendCount` : the number of people the Twitter user follows.
- `WriteGEXF(rdd: RDD[((String, String, String), Int)], gexfPath) : Boolean` and `WriteGraphML (rdd: RDD[((String, String, String), Int)], graphmlPath) : Boolean` : Output a GEXF or Graphml file from an output containing `((CrawlDate, Source, Destination), count)` values for quick visualization using network graph visualization tools like Gephi or SigmaJS.

### Image-Based Utilities

- `ComputerImageSize(bytes : Array[Byte]): (Int, Int)` : Used by `ExtractPopularImages` to calculate the size of the image as a tuple of integers (width, height).
- `ComputerMD5(bytes : Array[Byte])` : Computers the MD5 checksum of a byte array (eg. an image).  For string data, it is better to use `.computerHash()`.
- `ExtractImageLinks(src : String, html: String): Seq[String]` : Extracts image hrefs from html content and returns the full url by concatenating them with the source url.
- `ExtractPopularImages( records: RDD[ArchiveRecord], limit: Int, sc: SparkContent, minWidth: Int = 30, minHeight: Int = 30)` : Extract the n most popular images where n is equal to `limit` if the images have a width bigger than `minWidth` and a height bigger than `minHeight`.

## RDD functions

`import io.archivesunleashed.spark.rdd`

- `RecordRDD` : a class with functions used primarily for filtering data in RDDs, usually RDD[ArchiveRecord].
    * `.countItems()` : Count the matching items in the RDD and sort them in descending order.
    * `.keepValidPages()` : remove any non html-like pages.  Currently the default output of the RecordLoader.
    * `.keepImages()` : keep images in the RDD.
    * `.keepMimeTypes(mimeTypes: Set[String])` : keep the mime types (e.g. "text/html") recorded in `mimeTypes`.
    * `.keepDate(dates: List[String], component: DateComponent = DateComponent.YYYYMMDD)` : keep the dates listed in the `dates` list, in the format determined by DateComponent with YYYMMDD chosen if DateComponent is empty.
    * `.keepUrls(urls: List)` : keep only the urls listed in `urls`.
    * `.keepUrlPatterns(urlREs: Set[Regex])` : keep only the urls that match the regular expression patterns listed in `urlREs`.
    * `.keepDomains(urls: Set[String])` : keep only the source domains listed in `urls`.
    * `.keepLanguages(lang: Set[String])` : keep only the data in the listed languages.
    * `.keepContent(contentREs: Set[Regex])` : keep only the text that matches the regular expression patterns listed in `contentREs`.
    * `.discardMimeTypes(mimeTypes: Set[String])` : remove mime types listed in `mimeTypes`.
    * `.discardDate(date: String)` : remove data with the cited crawlDate `date`.
    * `.discardUrls(urls: Set[String])` : remove urls listed in `urls`.
    * `.discardUrlPatterns(urlREs: Set[Regex])` : remove urls that match the regular expressions found in `urlREs`.
    * `.discardDomains(urls: Set[String])` : discard data with the source domains listed in `urls`.
    * `.discardContent(contentREs: Set[Regex])` : discard content that matches the regular expressions listed in `contentREs`.

## Utilities

`import io.archivesunleashed.spark.utils`

- `JSONUtil` : Utilities for writing and parsing JSON strings.
    * `.toJson(value: Map[Symbol, Any] \{or Any\}): String` : Convert a Map or other date to Json format string.
    * `.fromJson(json: String): Map[String, Any]` : Ingest a Json String into a string map for easy parsing.
