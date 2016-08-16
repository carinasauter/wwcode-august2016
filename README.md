## Women Who Code East Bay big data workshop (August 17th, 2016)

For our project, we're going to build an anomaly detection system using Spark and Zeppelin.

The data we'll work on comes from UC Irvine, which has a great collection of already-cleaned machine-learning datasets. The example is based on an implementation in [Advanced Analytics with Spark](http://shop.oreilly.com/product/0636920035091.do) (highly recommended if you want to dig further into Spark).


## Some notes before we start

Spark and Zeppelin will be running as a separate, Dockerized server on your own laptop. This means:
- the speed of your analyses will depend on the speed of your laptop,
- your laptop might get really hot or drain its battery really fast,
- you can feel free to leave at any point and continue the analysis there (if you have to go), and
- after tonight, you'll have a full-powered, general-purpose big-data server you can use on any future problems.

We'll use 10% of a real data set so that our analyses run fast-ish, but after we're done feel free to download the larger version and run our whole analysis on that one instead.

The language we'll use to talk to Spark is Scala.

> Optional/explanatory text will be offset like this. You don't have to do anything with it, or even read it if you don't want to, though it may make things less confusing.


## Preparation (time-consuming downloads)

#### Step 0 (if you're on Mac OS X 10.10.3 or newer) — Install and run Docker for Mac

- Visit the [Docker for Mac](https://www.docker.com/products/docker#/mac) page and download Docker for Mac.
- Open the downloaded .dmg file and drag Docker to your Applications folder.
- Run Docker.
- Open a terminal window and type `docker run hello-world` to test whether your installation is working.
- If you see output that contains the line "Hello from Docker!", keep this terminal window open and proceed to step one.


#### Step 0 (if you're on Windows 10 Professional/Enterprise 64-bit) — Install and run Docker for Windows

- Visit the [Docker for Windows](https://www.docker.com/products/docker#/windows) page and download Docker for Windows.
- ...


#### Step 0 (if you're on Mac OS X 10.10.2 or older) — Install and run Docker Toolbox

- Visit the [Docker Toolbox](https://www.docker.com/products/docker-toolbox) page and download Docker Toolbox (Mac version).
- Open the downloaded .pkg file and follow the prompts to install the Toolbox.
- At the end of the install process, choose Docker Quickstart Terminal.
- This should open up a new terminal window and run through an installation script, ending with the terminal drawing an ASCII whale.
- In this terminal window, type `docker run hello-world` to test whether your installation is working.
- If you see output that contains the line "Hello from Docker!", keep this terminal window open and proceed to step one.


#### Step 0 (if you're on a version of Windows other than Windows 10 Professional/Enterprise 64-bit) — Install and run Docker Toolbox

- Visit the [Docker Toolbox](https://www.docker.com/products/docker-toolbox) page and download Docker Toolbox (Mac version).
- ...


#### Step 1 — Get Spark-and-Zeppelin image

Now that you have Docker installed and running, you can get a Linux virtual machine with Hadoop, Spark, and Zeppelin pre-installed running. Run this command in the terminal:

```bash
docker pull melindalu/zap
```

This will show a series of download and extraction steps.
These are the different components of the Spark/Zeppelin image being downloaded and assembled.
This requires a 777 MB download, so it may take a while.


#### Step 2 — Download our data set and move it to where we need it

- Visit UC Irvine's [KDD Cup 1999 data](http://kdd.ics.uci.edu/databases/kddcup99/kddcup99.html) page and download the data file `kddcup.data_10_percent.gz`.
- Create a new folder at path
    - `~/Downloads/zeppelin` (if you're on Mac OS X) or
    - `C:\Users\[username]\Downloads\zeppelin` (if you're on Windows).
- Remember this folder's path! We're going to reference it as `<your zeppelin-data folder path>` in step 3.
- Move the data file you've copied into that folder.

Great — preparation complete.


> ## What we're going to implement

> ### The problem

> We're going to use a really common unsupervised machine-learning technique called **clustering**. Often, we have a bunch of data that we know nothing about, and we want to find out what patterns there are in the data.
> Clustering is a way we can try finding natural groupings in data — by putting data points that are like each other, but unlike others, in the same cluster.

> Clustering lets us pinpoint data points that are anomalous, or out of the ordinary. For example, it can help us discover problems on servers or with sensor-equipped machinery, where we often need to detect failure modes we haven't seen before.

> We're going to use clustering to help detect network attacks. In a network, we have far too much traffic for a human to look at, and we don't always know what exploit an attacker might use to intrude. We want to be able to detect anomalous traffic. To do so, we need first to find our network data's natural groups.

> ### The algorithm

> *k*-means clustering is probably the most commonly-used clustering algorithm today. It tries to find *k* clusters in a data set, where *k* is chosen by the data scientist herself. The right value to choose for *k* depends on the data set, and finding a suitable value for *k* is a key part of the process.

> The *k*-means algorithm requires that we have a concept of what it means for data points to be "like" or "unlike" each other. We need a way to see if points are "near" or "far" from each other — that is, we need a notion of distance. Today, we're going to use simple Euclidean (i.e. straight-line) distance to measure distance between data points. (This will require our features to all be numeric.) The smaller the distance between two points, the more "alike" they are.

> To the *k*-means algorithm, a cluster is just a point — the center of all the points that make up the cluster. This center is called the *cluster centroid,* and is the (arithmetic) mean of the points in the cluster.

> #### How it works
> To start, the *k*-means algorithm randomly picks *k* data points as the initial cluster centroids, and every single data point in the set gets assigned to its nearest centroid. Next, for each cluster, a new cluster centroid is computed as the mean of the data points newly assigned to that cluster. This process is repeated until each iteration stops moving the centroids very much.


## Implementation

#### Step 3 — Run the Spark-and-Zeppelin image and check that it's working

- Open a terminal window. (For those using Docker Toolbox, this needs to be a Docker Quickstart Terminal window.)
- Run this command (with the `<your zeppelin-data folder path>` replaced with your folder from step 2):

```bash
docker create -v <your zeppelin-data folder path>:/var/zeppelin/data -p 8080:8080 melindalu/zap | xargs docker start -i
```

> ##### What's going on:
> - `docker create` creates a Docker container but doesn't start it.
> - `melindalu/zap` is the Docker image (already saved locally) that we're running.
> - The `-v <your zeppelin-data folder path>:/var/zeppelin/data` part makes your Zeppelin data folder accessible as a folder inside your Docker container.
> - The `-p 8080:8080` part makes the Zeppelin web server running on port 8080 in the Docker container accessible to your laptop.
> - `| xargs` pipes the result of `docker create` to `docker start`
> - `docker start -i` starts the image interactively — that is, with the container's command-line input available.

This will show a series of initialization steps as Spark and Zeppelin start up.
Now, open a web browser and go to URL `localhost:8080`.
If all is well, you should see a page welcoming you to Zeppelin.


#### Step 4 — Create a new note and check that it's working

A "note" is how we use Zeppelin to execute code in Spark. To create one, click `Create new note`, and name your note whatever you like — for example, "k-means".

When your note opens, it will have created your first "paragraph" (the block-looking thing).  
Replace what's in it with

```scala
%spark
println("testing testing 1-2-3")
```

and press the Play button or hit `Shift+Enter` to run the paragraph.  
After a long pause (during which your computer is submitting your code to Spark), the browser should print your message — if it does, you'll know Spark is alive and waiting.


#### Step 5 — Unzip your data and load it into Spark

Now replace everything in your first paragraph with

```bash
%sh
gunzip /var/zeppelin/data/kddcup.data_10_percent.gz
```

> ##### What's going on:
> The `%sh` tells Zeppelin to submit your command to the shell instead of Spark, and `gunzip` unzips our data file.

If this completes happily, this means you've successfully loaded your dataset where Spark can reach it — often, this is the hardest part of a big data problem.


#### Step 6 — Load our data into Spark and inspect it a little

Create a new paragraph (by hovering over the bottom border of your last paragraph until you see the + and clicking).
In the new block, paste in:

```scala
%spark
val rawData = sc.textFile("/etc/zeppelin/data/kddcup.data")
rawData.take(10).foreach(println)
rawData.count()
```

and run the block (by hitting the Play button or `Shift+Enter`).

This loads the data file into Spark as the variable `rawData`, prints the first 10 records so we can see what our data looks like, and counts the total number of records in our data set.

As you can see, each record is a string of comma-separated data, containing 38 features. Some features are counts, many features have value either 0 or 1, and a category is given in the last field. We're not going to use the categories to help with clustering, but we can look at them before we start to get an idea of what to expect.


#### Step 7 — Tally up how many of each label there are

Create another new block below.

Begin by exploring the data set. What categories are present in the data, and how many data points are there in each category? Paste in and run the following code to see:

```scala
%spark
val labelCounts = rawData.map(_.split(',').last).countByValue().toSeq.sortBy(_._2).reverse
labelCounts.foreach(println)
```

This splits off the label, counts up total number of records per label, sorts this descending by count, and prints the result. We can see there are 23 distinct labels, and the most frequent are `smurf.` and `neptune.` attacks.


#### Step 8 — Maybe that would look better as a graph

The Zeppelin graphing language is a little weird, but let's try it out.  
Create a new block, and paste in and run:

```scala
%spark
println("%table label\tcount")
labelCounts.foreach{ case (label, count) => println(label + "\t" + count)}
```

This should have autocreated a little pop-out row of icons.  
Click on the various icons to see how Zeppelin wants to help us visualize the data — I'd suggest the bar chart, or the line chart.

Okay, enough poking around — let's start our *k*-means clustering.


#### Step 9 — Prepare to make a first pass at clustering

Right now, our data contains some nonnumeric features — for example, the second column may be tcp, udp, or icmp, and the final column (which we just explored) is a nonnumeric category label. *k*-means clustering requires numeric features, so for now, we'll just ignore the nonnumeric fields.

In another new block, paste in and run:

```scala
%spark
import org.apache.spark.mllib.linalg._

val labelsAndData = rawData.map { line =>
  val buffer = line.split(',').toBuffer // toBuffer creates Buffer, a mutable list
  buffer.remove(1, 3)
  val label = buffer.remove(buffer.length - 1)
  val vector = Vectors.dense(buffer.map(_.toDouble).toArray)
  (label,vector)
}

val preparedData = labelsAndData.values.cache()
```

This splits the comma-separated-value strings into columns, removes the three categorical value columns at indices 1-3, and removes the final column. The remaining values are converted to an array of `Double`s and emitted with the final label column in a tuple.


#### Step 10 — Machine-learning time: first pass at *k*-means clustering

*k*-means is built into the Spark MLLib standard library, so clustering our data is as simple as importing the `KMeans` implementation and running it.

The following code clusters the data to create a `KMeansModel` and then prints its centroids. Create a new block, paste it in, and run:

```scala
%spark
import org.apache.spark.mllib.clustering._

val kmeans = new KMeans()
val firstModel = kmeans.run(preparedData)

firstModel.clusterCenters.foreach(println)
```

Your computer will crunch away (slowly), doing some serious machine learning.

And very unassumingly, when it's done, it'll print out two vectors. These vectors are the centroids of the two clusters Spark has chosen — meaning that *k*-means was fitting k = 2 clusters to the data.
For a complex data set (that we secretly know has at least 23 distinct types of connections), this is almost certainly not enough to accurately model the distinct groupings within the data.


#### Step 11 — See how well we did in our first pass

This is a good place for us to use the given categories to get an idea of what went into these two clusters — we can look at the categories that ended up in each cluster.

Create a new block, paste in the following code, and run. This assigns every data point to one of the two clusters using the model, counts up how many points in each category are in each cluster, then shows this in a table or graph.

```scala
%spark
val clusterLabelCount = labelsAndData.map { case (label, datum) =>
  val cluster = firstModel.predict(datum)
  (cluster,label)
}.countByValue

println("%table cluster\tlabel\tcount")
clusterLabelCount.toSeq.sorted.foreach {
  case ((cluster, label), count) =>
    println(s"$cluster\t$label\t$count")
}
```

The result shows that the clustering was pretty unhelpful — how many points ended up in each cluster?


#### Step 11 — This time, let's choose a better *k* (with math)

So two clusters aren't enough — how many clusters should we choose for this data set? We know that there are 23 distinct patterns in the data, so it seems that *k* could be at least 23 — probably even more. Typically, a data scientist will try many values of *k* in order to find the best one. How does she define "best?

A clustering could be considered better if its data points were closer to their respective centroids. To keep track of our distances, let's define a Euclidean distance function and a function that returns the distance from a data point to its nearest cluster's centroid. In a new paragraph, paste in and run:

```
def euclideanDistance(a: Vector, b: Vector) =
  math.sqrt(a.toArray.zip(b.toArray).
    map(p => p._1 - p._2).map(d => d * d).sum)

def distanceToCentroid(datum: Vector, model: KMeansModel) = {
  val cluster = model.predict(datum)
  val centroid = model.clusterCenters(cluster)
  euclideanDistance(centroid, datum)
}
```

You can read off the definition of Euclidean distance here by unpacking the Scala function, in reverse: sum (sum) the squares (map(d => d * d)) of differences (map(p => p._1 - p._2)) in corresponding elements of two vectors (a.toArray.zip(b.toArray)), and take the square root (math.sqrt).


#### Step 12 — More math towards a better *k*

Using the above, we can define a scoring function that measures the average distance to centroid for a model built with a given *k*. Create a new paragraph, paste in, and run:

```scala
%spark
import org.apache.spark.rdd._

def clusteringScore(data: RDD[Vector], k: Int) = {
  val kmeans = new KMeans()
  kmeans.setK(k)
  val model = kmeans.run(data)
  data.map(datum => distanceToCentroid(datum, model)).mean()
}

(5 to 40 by 5).map(k => (k, clusteringScore(data, k))).
  foreach(println)
```

Here we're written our scoring function and are using it to evaluate values of *k* from 5 to 40.

> `(x to y by z)` is a Scalaism for creating a collection of numbers between a start and end, inclusive, with a given difference between successive elements. This is a concise way to create the values `k = 5, 10, 15, 20, 25, 30, 35, 40` then do something with each.

The result should show that the score decreases as *k* increases.


#### Step 13 — Plot our results for different *k*s

```scala
%spark
<TODO>
```

#### Step 14 — Redo clustering with our new, better *k*

```scala
%spark
<TODO>
```

#### Step 15 — Visualize our clusters

```scala
%spark
<TODO>
```

## Where to next

```
Feature normalization
Putting our categorical variables back in
Better measures of cluster quality
Streaming KMeans
Different models
```
