---
layout: post
title: Getting started with Spark and Zeppellin
---

This is a presentation I prepared for the January 2016's [Montreal Apache Spark Meetup](http://www.meetup.com/Montreal-Apache-Spark-Meetup/events/227554437/).
It was originally a [Zeppelin](https://zeppelin.incubator.apache.org) notebook that I turned into this blog post.  Currently, Zeppelin notebooks can only be exported as `json` files, so I wrote a very simple [python script](https://github.com/jfraj/zeppelinConverter) to turn it into a markdown file that I tweaked for this blog post.
The purpose of this presentation was to provide a first contact with Spark,
so I start with a scope warning:

### In scope
* The very basic stuff
* Install locally on a mac.
* Show how to read csv data
* First steps of exploring data

### Out of scope
* cluster settings


Most of the examples are in scala, but things are simple enough that it should be understandable even if you do not have experience with it.

## Local Installation
These are very limited instructions, they worked for me, but I had most of the tools installed.  During the meetup, many attendees had to install missing tools like `brew` and `npm`.

* Download pre-build from [http://spark.apache.org/downloads.html](http://spark.apache.org/downloads.html)
* Choose newest pre-build, I used `(1.5.2) pre-build for hadoop 2.6 and later` but I also tried succesfully 1.6 over the weekend.
* Untar wherever you like (I did `$HOME/local_spark/`)
* Test the shell by typing (in the installed directory) `./bin/spark-shell`
    * `--master` to set the master
        * `local` locally,  one worker thread
        * `local[n]` locally, `n` worker thread (`n= * ` mean # of core available)
* More tests are suggested in the [documentation](http://spark.apache.org/docs/latest/)

## Spark Shell
The Spark Shell offers the usual REPL (Read Evaluate Print Loop) features. The most common shell flavors are

* **scala** The native shell `./bin/spark-shell` (a modified version of the Scala shell)
* **pyspark** `./bin/pyspark` to use all things you love from Python.
    * **Note**: Pyspark is also accessible in the Python interpreter, all you need to do is set these environment variables (where `$SPARK_HOME` is where you installed spark)

~~~
    export PYTHONPATH=$SPARK_HOME/python:$SPARK_HOME/python/build:$PYTHONPATH
    export PYTHONPATH=$SPARK_HOME/python/lib/py4j-0.8.2.1-src.zip:$PYTHONPATH
~~~

### Spark Context
Spark context is your connection to the spark cluster.  When you start the spark shell, you get the message `Spark context available as sc`. The spark context allows to

* Run jobs
* Accessing service s.a.  Task scheduler
* Serialize data, like creating RDD (Resillient Distributed Datasets), in other words, distribute data on your cluster.
* ...

## Notebooks
For richer interactive sessions, reproducibility and sharing, you can chose different version of notebooks.  They use the power of your web browser for interactive display. A well known example is Jupyter (formerly known as ipython notebook) used, among others, in the python community.

* **Zeppelin** used here
    * built on JVM
    * Intended for Spark
    * Apache incubator
* **Jupyter** (not tried)
    * Very mature
    * huge community
* **Databricks**
    * Great (I use it at work)
    * Commercial $$$
* **More**
    * [Beaker notebook](http://beakernotebook.com/)
    * [spark-notebook](https://github.com/andypetrella/spark-notebook/)
    * ...

### Zeppelin
easy install with [Apache Maven](https://maven.apache.org/install.html).

* download `git clone https://github.com/apache/incubator-zeppelin/`
* `cd incubator-zeppelin/`
* `mvn clean package -Pspark-1.5 -DskipTests`
* `./bin/zeppelin-daemon.sh start`
* on your favorite browser, go to `http://localhost:8080/`

The rest of this post shows input-output as given by zeppelin in this format
{% highlight scala %}
code input
{% endhighlight %}
<blockquote>
Text output
</blockquote>
For markdown-friendliness, some characters like `<` have been removed from the output.

The inspiration to look at airbnd data came from this nice [blog entry](http://www.racketracer.com/2015/12/28/getting-started-apache-zeppelin-airbnb-visuals/).

##RDD
Resillient Distributed Datasets are the basic data abstractions in Spark.
They are immutable and distributed over (partioned across) the available cluster.
Calculation can be performed in parallel over the partitioned elements.

####RDD can be created from a sequence of elements.
{% highlight scala %}
val wordsRDD = sc.parallelize(List("fish", "cats", "poutine"))
{% endhighlight %}

<blockquote>
wordsRDD: org.apache.spark.rdd.RDD[String] = ParallelCollectionRDD[83] at parallelize at console:23
</blockquote>

We can look at the `n` first elements of `wordsRDD` with the `take(n)` method
{% highlight scala %}
wordsRDD.take(2)
{% endhighlight %}

<blockquote>
res3: Array[String] = Array(fish, cats)
</blockquote>

#### Digression : Execution of external process
Below, we will use `import sys.process._` to run shell commands outside of spark in order to fetch data.  Obvisouly, these are system specific and below is to access [Airbnb data](http://insideairbnb.com/get-the-data.html) from a MAC.  Note the `!` at the end of a command string.

{% highlight scala %}
import sys.process._
"curl -O http://data.insideairbnb.com/canada/qc/montreal/2015-10-02/visualisations/listings.csv"!
"mkdir -p data"!
"mv listings.csv data/mtl_listings.csv"!
{% endhighlight %}

<blockquote>
import sys.process._ <br>
warning: there were 1 feature warning(s); re-run with -feature for details<br>
res32: Int = 0 warning: there were 1 feature warning(s); re-run with -feature for details<br>
res33: Int = 0 warning:there were 1 feature warning(s); re-run with -feature for details<br>
res34: Int = 0
</blockquote>
The file `mtl_listings.csv` is now in the `data/` subdirectory.


#### Reading from a text file
Here is a way to load the file as an RDD
{% highlight scala %}
val linesRDD = sc.textFile("data/airbnb/mtl_listingsWRONGNAME.csv")
{% endhighlight %}

<blockquote>
linesRDD: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at console:25
</blockquote>

All good! let's see how many elements there are by counting the lines.

{% highlight scala %}
linesRDD.count
{% endhighlight %}

<blockquote>
org.apache.hadoop.mapred.InvalidInputException: Input path does not
exist: file:data/mtl_listingsWRONGNAME.csv
...(more error message)
</blockquote>

It fails!  What just happen shows the lazy evaluation: as long as no action is ran (here `count`),
Spark does not process the data.  Hence the `does not exists` error not showing before.  Let's doing again with the right name.

{% highlight scala %}
val linesRDD = sc.textFile("data/mtl_listings.csv")
{% endhighlight %}

<blockquote>
linesRDD: org.apache.spark.rdd.RDD[String] = MapPartitionsRDD[3] at textFile at console:25
</blockquote>

{% highlight scala %}
linesRDD.count
{% endhighlight %}

<blockquote>
res7: Long = 8986
</blockquote>

All right. Let's now check the first three lines
{% highlight scala %}
linesRDD.take(3).foreach {println}
{% endhighlight %}

<blockquote>
id,name,host_id,host_name,neighbourhood_group,neighbourhood,latitude,longitude,room_type,price,minimum_nights,number_of_reviews,last_review,reviews_per_month,calculated_host_listings_count,availability_365<br>
<nobr>7771742,Cozy and Bright Room in Mile-end,40880361,Aletha,,Outremont,45.52230324180573,-73.60671958096545,Private room,37,1,1,2015-08-27,0.81,1,365</nobr>
<nobr>959727,Superbe appartement à Montréal,1792119,Christine,,Outremont,45.5197179590061,-73.61050026481905,Entire home/apt,78,21,0,,,1,122</nobr>
</blockquote>

That's long but it looks like a user-friendly csv file that we can deal with.

## Reading data
Spark SQL provides built-in support for 3 types of data sources:

* Parquet (default)
* Json
* Jdbc

example:
`val people = sqlContext.read.json("people.json")`


**Warning**: from [spark docs](https://spark.apache.org/docs/latest/sql-programming-guide.html#json-datasets) (about json):
Each line must contain a separate, self-contained valid JSON object. As a consequence, a regular multi-line JSON file will most often fail.

### The cheating way
One can use non-spark tool, like `pandas` to read the data and turn it into a spark-native format.<br>
Caveat: limited by memory.

##csv
I show two examples to acces the elements of the csv file.  If you do not have experience with the scala language, that might not be straightforward but that's ok: The point is to show that it is not simple so you can skip to the `Spark packages` section.

###Brute force: using case class
By looking at the rows, one can define a `case class` where you infer the type.
When in doubt, use `String`.
We can take any subsample of the fields.
{% highlight scala %}
case class Listing(id: Integer,name: String, neighbourhood: String, latitude: Double, longitude: Double)
{% endhighlight %}

<blockquote>
defined class Listing
</blockquote>

{% highlight scala %}
val listings = linesRDD.map(s => s.split(",")).filter(s => s(0) != "id").map(
    s => Listing(s(0).toInt,
            s(1),
            s(5),
            s(6).toFloat,
            s(7).toFloat
        )
)
{% endhighlight %}
<blockquote>
listings: org.apache.spark.rdd.RDD[Listing] = MapPartitionsRDD[1106] at map at console:27
</blockquote>

Let's see what we got
{% highlight scala %}
listings.take(3).foreach {println}
{% endhighlight %}
<blockquote>
<nobr>Listing(7771742,Cozy and Bright Room in Mile-end,Outremont,45.52230453491211,-73.60671997070312)</nobr>
<nobr>Listing(959727,Superbe appartement à Montréal,Outremont,45.519718170166016,-73.6104965209961)</nobr>
<nobr>Listing(6588054,Cosy & elegant apt (Outremont),Outremont,45.50696563720703,-73.61654663085938)</nobr>
</blockquote>

### Automize access csv-to-rdd fields
If you want to ease the access the the elements, you can create a map.
{% highlight scala %}
val headersAndRows = linesRDD.map(line => line.split(",").map(_.trim))
val header = headersAndRows.first
val data = headersAndRows.filter(_(0) != header(0))
val maps = data.map(splits => header.zip(splits).toMap)
{% endhighlight %}

<blockquote>
headersAndRows: org.apache.spark.rdd.RDD[Array[String]] = MapPartitionsRDD[1097] at map at console:26<br>
header: Array[String] = Array(id, name, host_id, host_name, neighbourhood_group, neighbourhood, latitude, longitude, room_type, price, minimum_nights, number_of_reviews, last_review, reviews_per_month, calculated_host_listings_count, availability_365)<br>
data: org.apache.spark.rdd.RDD[Array[String]] = MapPartitionsRDD[1098] at filter at console:29<br>
maps:<br> org.apache.spark.rdd.RDD[scala.collection.immutable.Map[String,String]] = MapPartitionsRDD[1099] at map at console:31
</blockquote>

You can then filter on a particular field
{% highlight scala %}
val results = maps.filter(map => map("id") == "6588054")
results.take(5).foreach(println)
{% endhighlight %}

<blockquote>
results: org.apache.spark.rdd.RDD[scala.collection.immutable.Map[String,String]] = MapPartitionsRDD[1100] at filter at console:33<br>
Map(<br>
neighbourhood_group -> , name -> Cosy & elegant apt (Outremont),<br>
room_type -> Entire home/apt, latitude -> 45.5069662162218,<br>
price -> 110, host_name -> Franck, longitude -> -73.61655040181864,<br>
calculated_host_listings_count -> 1, availability_365 -> 122,<br>
id -> 6588054, number_of_reviews -> 7, host_id -> 10810386,<br>
last_review -> 2015-08-30, reviews_per_month -> 1.79, minimum_nights -> 3<br>,
neighbourhood -> Outremont)
</blockquote>

### Spark packages
Reading a csv file is so common that a utility has been created
and comes as a `package`.
Packages are community contribution to spark, you can find them listed [here](http://spark-packages.org/).
More information can be found in databricks' package announcement [blog entry](https://databricks.com/blog/2014/12/22/announcing-spark-packages.html).

#### Packages in zeppelin
To use a package in zeppelin, you need to load it in the first cell of the notebook.
This works
{% highlight scala %}
z.reset()
// Add spark-csv package
z.load("com.databricks:spark-csv_2.10:1.3.0")
{% endhighlight %}


#### Package example [spark-csv](https://github.com/databricks/spark-csv)
You can also invoke a package when you initiate a spark shell

```
$SPARK_HOME/bin/spark-shell --packages com.databricks:spark-csv_2.10:1.3.0
```

We can now use spark-csv to read a csv file

{% highlight scala %}
val df = sqlContext.read
    .format("com.databricks.spark.csv")
    .option("header", "true") // Use first line of all files as header
    .option("inferSchema", "true") // Automatically infer data types //.option("quote", "\"")
    .option("mode", "DROPMALFORMED")
    .load("data/airbnb/mtl_listings.csv")
{% endhighlight %}

<blockquote>
df: org.apache.spark.sql.DataFrame =<br>
[id: string, name: string, host_id: string, host_name: string,<br>
neighbourhood_group: string, neighbourhood: string, latitude: double,<br>
longitude: string, room_type: string, price: int, minimum_nights: int<br>,
number_of_reviews: int, last_review: string, reviews_per_month: double<br>,
calculated_host_listings_count: int, availability_365: int]
</blockquote>

Looks good!
Whatever that thing we produced, it seems to have correctly inferred the type of the field,
e.g. setting `latitude` as `double`.
The output also tells that the produced object is a `org.apache.spark.sql.DataFrame`.

##DataFrames
Introduced in spark 1.3, Dataframes are build on top of RDDs.
They are the "Prefered abstraction in Spark" because they are

* Simpler
* Better optimized (especially for python, R... any non-scala)

Spark dataframes are inspired by R and Pandas dataframes **but immutable**.

###Creation
Dataframes can be created

* by parallelizing existing collections (e.g. RDDs, Pandas DataFrames)
* by transforming and existing DataFrame
* from files in storage system (e.g. parquet files)

### Schema
One can inspect the structure of a dataframe through the schema method
{% highlight scala %}
df.printSchema
{% endhighlight %}

<blockquote>
root<br>
 |-- id: string (nullable = true)<br>
 |-- name: string (nullable = true)<br>
 |-- host_id: string (nullable = true)<br>
 |-- host_name: string (nullable = true)<br>
 |-- neighbourhood_group: string (nullable = true)<br>
 |-- neighbourhood: string (nullable = true)<br>
 |-- latitude: double (nullable = true)<br>
 |-- longitude: string (nullable = true)<br>
 |-- room_type: string (nullable = true)<br>
 |-- price: integer (nullable = true)<br>
 |-- minimum_nights: integer (nullable = true)<br>
 |-- number_of_reviews: integer (nullable = true)<br>
 |-- last_review: string (nullable = true)<br>
 |-- reviews_per_month: double (nullable = true)<br>
 |-- calculated_host_listings_count: integer (nullable = true)<br>
 |-- availability_365: integer (nullable = true)<br>
</blockquote>

Here is a filter and select command that shows how they are user-friendly
{% highlight scala %}
df.filter("id = 6588054").select($"id", $"name", $"neighbourhood").show
{% endhighlight %}

~~~
+-------+--------------------+-------------+
|     id|                name|neighbourhood|
+-------+--------------------+-------------+
|6588054|Cosy & elegant ap...|    Outremont|
+-------+--------------------+-------------+
~~~
The above output is reminiscent to sql output.
Indeed, sql and spark have many things in common.


## Spark SQL
Just like there is a spark context, there also is a sql context.
You can have two types

* SQLContext (default) support a “rich subset dialect of SQL 92”
* HiveContext "dialect is hiveql" richer than SQLContext

Let's create a table by registering the dataframe

{% highlight scala %}
df.registerTempTable("airbnb")
{% endhighlight %}

You can run sql query using the `sqlContext`
{% highlight scala %}
sqlContext.sql(
    """
        SELECT name, price
        FROM airbnb
        LIMIT 5
    """
).show
{% endhighlight %}

~~~
+--------------------+-----+
|                name|price|
+--------------------+-----+
|Cozy and Bright R...|   37|
|Superbe apparteme...|   78|
|Cosy & elegant ap...|  110|
|Spacious 2 bedroo...|   90|
|Beautiful typical...|  116|
+--------------------+-----+
~~~
That is helpful to recycle your sql knowledge from a previous life.

Interesting facts about registered tables, their performance is

* the same performance as DataFrames
* better performance as RDDs

In zeppelin you can set the cell to accept pure sql query
{% highlight sql %}
%sql
SELECT AVG(reviews_per_month) monthly_reviews, neighbourhood
FROM airbnb
GROUP BY neighbourhood
ORDER BY monthly_reviews DESC
LIMIT 2
{% endhighlight %}

<blockquote>
monthly_reviews	neighbourhood<br>
2.1234569536423855	Ville-Marie<br>
1.8704347826086958	LaSalle<br>
</blockquote>
Finally, one great feature of notebooks is the visual output. Instead of having the values listed, they can be plotted.
Here is a screenshot of the same query (with a higher `LIMIT`) showing the output as a bar chart.
![png]({{jfraj.github.io}}/assets/gettingStartedWithSpark_files/zepSqlAirBnb.png)
