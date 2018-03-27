----
-title: "Archives Unleashed Toolkit Library"
-date: 2018-02-28T14:57:14-05:00
-draft: false
----

# Archives Unleashed Toolkit Library

The Archives Unleashed Toolkit (AUT) is a collection of tools for filtering, aggregating, analyzing and
visualizing web archive data. The toolkit is contained in four repository folders: The Archive Record Class, Matchbox, Resilient Distributed Dataset ([RDD](https://spark.apache.org/docs/latest/rdd-programming-guide.html) Tools, and Utilities.

## The Archive Record Class

`import io.archivesunleashed.spark.archive.io.ArchiveRecord`

`class ArchiveRecord(r: SerializableWritable[ArchiveRecordWritable]) extends Serializable`

### Description

The `ArchiveRecord` [class](https://docs.scala-lang.org/tour/classes.html) is used by `RecordLoader` to manage the ingestion of WARC and ARC files. It represents one record from a collection found in the file. It uses the Internet Archive's ARCRecord & WARCRecord APIs to extract data from web crawls such as the crawl date, content strings, urls and so on.

`ArchiveRecord` is a generic record that covers data from both ARCRecord and WARCRecord format. AUT will convert either format into the generic `ArchiveRecord` class.

### Variables

- `.arcRecord: ARCRecord` : if one is detected, stores a record from an ARC File.
- `.warcRecord: WARCRecord` : if one is detected, stores a record from a WARC File.
- `.ISO8601: SimpleDateFormat` : Format for CrawlDates. `\"yyyy\-MM\-dd\'T\'HH:mm:ssX\` by default.
- `.getCrawlDate: String` : Extract the CrawlDate from the record with format `YYYYMMDD`.
- `.getCrawlMonth: String` : Extract the CrawlDate from the record with format `YYYYMM`.
- `.getContentBytes: Array[Byte]` : Extract the (HTML) content as a byte array.
- `.getContentString: String` : Extract the (HTML) content as a string.
- `.getMimeType: String` : Extract the MimeType (e.g. `text/HTML` or `video/mp4`).
- `.getUrl : String` : Extract the source url (where the crawl occurred).
- `.getDomain : String` : Extract the source domain (see ExtractDomain).
- `.getImageBytes : Array[Byte]` : Extract Image data as a byte array.

## The Archives Unleashed Matchbox

`import io.archivesunleashed.spark.matchbox`

### Description

The "matchbox" contains a collection of "Spark" tools to ignite the data that comes from
one or more ArchiveRecords. The most important tool is the `RecordLoader` which takes a
file or files in a path and returns a RDD of `ArchiveRecord`.

### Loading ArchiveRecords

- `RecordLoader` : The RecordLoader is usually the first call when using AUT. It loads a WARC or ARC file and puts it into a Spark RDD for quick iterating and derivative creation.
    * `loadArchives(path: String, sc: SparkContext, keepValidPages: Boolean = true): RDD[ArchiveRecord]` : Load a WARC or ARC into Spark. By default, it will remove any invalid pages from the analysis, unless keepValidPages is false.
    * `loadTweets(path: String, sc: SparkContext): RDD[JValue]` : Load JSON from a file that contains Twitter API data and store it in an RDD. Uses `json4s` and `json4s.jackson.JsonMethods`.

### Text Utilities

- `DetectLanguage(input:String): String` : Detects the language of a text input. Returns the [ISO639-2 language code] (https://www.loc.gov/standards/iso639-2/php/code_list.php) for the detected language ('en' for English, 'fr' for French etc.).
- `DetectMimeTypeTika(content: String): String` : Detects the Mime Type (e.g. `text/HTML`) using Tika.
- `ExtractBoilerpipeText(String)` : Extracts & removes boilerplate text (e.g. a generic copyright statement)
from website content using [Boilerpipe](https://boilerpipe-web.appspot.com/).
- `ExtractAtMentions(String): List[String]` : Extracts all `@username` from a Twitter status update or other content string, returning a list of users.
- `ExtractDate(fullDate :String, dateFormat: DateComponent): String` : Extract a date from a string in the desired format (`YYYY`, `MM`, `DD`, `YYYYMM`, or `YYYYMMDD`).
- `ExtractDomain(url: String, source: String=""): String` : Extracts the domain host (`www.example.com`)
from a full URL (`www.example.com/additional/file/path/info`). If desired, an additional string
can be passed to the function as a default for the case the URL input returns `null`.
- `ExtractEntities
    * (iNerClassifierFile: String, inputRecordFile: String, outputFile: String, sc: SparkContext)` : Conduct [Named Entity Recogition](https://nlp.stanford.edu/software/CRF-NER.html) classification on a WARC or ARC record using the path to a [NER classifier](https://stanfordnlp.github.io/CoreNLP/), an ARC or WARC file path, an output file and a Spark context.
- `ExtractHashtags(src: String): List[String]` : Extract all `#hashtags` from a Twitter status update or other content string, returning a list of hashtags.
- `ExtractLinks(src: String): List[String]` : Extract all URLs from a content string, returning a list of detected URLs.
- `RemoveHTML (content : String)` : Removes HTML tags from content data.
- `RemoveHttpHeader ( content: String): String` : Removes the http header (`http://` or `https://`) from a URL string.
- `StringUtils` : Tools for modifying strings.
    * `.removePrefixWWW() : String` : Removes the `www`. from a URL if it contains one. This avoids duplication of websites (`www.example.com` vs `example.com`).
    * `.escapeInvalidXML() : String` : Converts invalid XML tags `\&`, `\"`, `\<`, `\>` to XML-safe entities (&amp;, &quot;, &lt;, &gt;). May be able to resolve some issues due to corrupt data.
    * `.computeHash() : String` : Converts a string to an MD5 checksum. Used to create ids in GEXF or GraphML outputs, but can also be used to anonymize data.
- `TweetUtils` : Tools for manipulating Twitter API JSON Data.
    * `.id()` : Extract the Twitter id.
    * `.createdAt()` : Get the date a tweet was posted.
    * `.text()` : Get the text from the tweet.
    * `.lang` : Get the language.
    * `.username()` : Get the username of the person who posted the tweet.
    * `.isVerifiedUser` : `true` if the user is a "Verified User" otherwise, `false`.
    * `.followerCount` : The number of followers for the Twitter user.
    * `.friendCount` : The number of people the Twitter user follows.
- `WriteGEXF(rdd: RDD[((String, String, String), Int)], gexfPath) : Boolean` : Output a GEXF file from an output containing `((CrawlDate, Source, Destination), count)` values for import into network graph visualization tools like Gephi or SigmaJS.
- `WriteGraphML (rdd: RDD[((String, String, String), Int)], graphmlPath) : Boolean` : Output a GraphML file from an output containing `((CrawlDate, Source, Destination), count)` values for import into network graph visualization tools like Gephi or SigmaJS.

### Image-Based Utilities

- `ComputeImageSize(bytes : Array[Byte]): (Int, Int)` : Used by `ExtractPopularImages` to calculate the size of the image as a tuple of integers (width, height).
- `ComputeMD5(bytes : Array[Byte])` : Computes the MD5 checksum of a byte array (eg. an image). For string data, it is better to use `.computeHash()`.
- `ExtractImageLinks(src : String, HTML: String): Seq[String]` : Extracts image hrefs from HTML content and returns the full URL by concatenating them with the source URL.
- `ExtractPopularImages( records: RDD[ArchiveRecord], limit: Int, sc: SparkContent, minWidth: Int = 30, minHeight: Int = 30)` : Extract the n most popular images where n is equal to `limit` if the images have a width bigger than `minWidth` and a height bigger than `minHeight`.

## RDD functions

`import io.archivesunleashed.spark.rdd`

- `RecordRDD` : A class with functions used primarily for filtering data in RDDs, usually RDD[ArchiveRecord].
    * `.countItems()` : Count the matching items in the RDD and sort them in descending order.
    * `.keepValidPages()` : Remove any non HTML-like pages. Currently the default output of the RecordLoader.
    * `.keepImages()` : Keep images in the RDD.
    * `.keepMimeTypes(mimeTypes: Set[String])` : keep the mime types (e.g. `text/HTML`) recorded in `mimeTypes`.
    * `.keepDate(dates: List[String], component: DateComponent = DateComponent.YYYYMMDD)` : Keep the dates listed in the `dates` list, in the format determined by DateComponent with YYYMMDD chosen if DateComponent is empty.
    * `.keepUrls(urls: List)` : Keep only the URLs listed in `urls`.
    * `.keepUrlPatterns(urlREs: Set[Regex])` : Keep only the urls that match the regular expression patterns listed in `urlREs`.
    * `.keepDomains(urls: Set[String])` : Keep only the source domains listed in `urls`.
    * `.keepLanguages(lang: Set[String])` : Keep only the data in the listed languages.
    * `.keepContent(contentREs: Set[Regex])` : Keep only the text that matches the regular expression patterns listed in `contentREs`.
    * `.discardMimeTypes(mimeTypes: Set[String])` : remove mime types listed in `mimeTypes`.
    * `.discardDate(date: String)` : Remove data with the cited crawlDate `date`.
    * `.discardUrls(urls: Set[String])` : Remove URLs listed in `urls`.
    * `.discardUrlPatterns(urlREs: Set[Regex])` : Remove urls that match the regular expressions found in `urlREs`.
    * `.discardDomains(urls: Set[String])` : Discard data with the source domains listed in `urls`.
    * `.discardContent(contentREs: Set[Regex])` : Discard content that matches the regular expressions listed in `contentREs`.

## Utilities

`import io.archivesunleashed.spark.utils`

- `JSONUtil` : Utilities for writing and parsing JSON strings.
    * `.toJson(value: Map[Symbol, Any] \{or Any\}): String` : Convert a Map or other date to JSON format string.
    * `.fromJson(json: String): Map[String, Any]` : Ingest a JSON String into a string map for easy parsing.
