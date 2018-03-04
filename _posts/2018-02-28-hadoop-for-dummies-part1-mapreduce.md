---
author: Ilyes
layout: post
title:  "Hadoop Ecosystem For Dummies | Part 1 (MapReduce)"
date:   2018-02-28
comments: true
category:
- Hadoop
- MapReduce
---

MapReduce is a programming model and an associated implementation for processing and generating big data sets with a parallel, distributed algorithm on a cluster. [" wiki "](https://en.wikipedia.org/wiki/MapReduce "Wikipedia")

## Prerequisite
As we make always sure that this blog is not about theories and long explanations, so I will claim that you already know what a [Hadoop Ecosystem](https://hadoopecosystemtable.github.io/) is? [HDFS](https://wiki.apache.org/hadoop/HDFS)? How MapReduce logic works and that you're familiar working with the [CLI](https://en.wikipedia.org/wiki/CLI) and Linux commands.
If not, please make sure to refer to the links sources.

## Study case
Today, we will try to discover how one of Hadoop's principal components **MapReduce** works, so we will work in a very simple real-world problem.

Let's imagine that we run a hypermarket with hundreds of thousands of stores around the world, say Cora for instance, and that we have a very big log file of all the sales for 2017, and we want to calculate the total of sales per store.

Let's say that our data is structred in this way
<table style="margin: auto;" >
<tr><th>Date of Purchase</th><th>Hour of purchase</th><th>Store Location</th><th>Product</th><th>Amout</th><th>Payment method</th></tr>
<tr><td>2017-02-05</td><td>11:23</td><td>London</td><td>Pet supplies</td><td>$18.00</td><td>Master</td></tr>
<tr><td>2017-02-05</td><td>11:23</td><td>Paris</td><td>Camera</td><td>$220.00</td><td>Visa</td></tr>
<tr><td>2017-02-04</td><td>11:23</td><td>NYC</td><td>Hand Watch</td><td>$40.95</td><td>Master</td></tr>
<tr><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td><td>...</td></tr>
</table><br>


One of the traditional ways to solve this kind of problem is by using a Hashtable, where data is presented in a key-value way, the keys are the names of the stores, and the values are the total sales per store.

Say we have three terabytes of data, so with this approach, we may run out of memory and the data will certainly take a long time to be all processed in a sequence way.

## Setting the environment
For the implementation, we'll work on a pre-installed 100% open source platform distribution from Cloudera called CDH [(Cloudera's Distribution for Hadoop)](https://www.cloudera.com/products/open-source/apache-hadoop/key-cdh-components.html)
It's a Virtual Machine, so you need first to download a Virtual Machine workspace like [VMware](https://my.vmware.com/fr/web/vmware/downloads) or [VirtualBox](https://www.virtualbox.org/wiki/Downloads), then you [download the CDH](https://www.cloudera.com/downloads.html), you open it up, start the VM, and you will have something similar to this screenshot.

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/CDH_home.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/CDH_home.jpg "Figure1. CDH CLI")

## Workspace
First, let's see what we have in the local machine directory...

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/ls_command.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/ls_command.jpg "Figure2. local machine directory content")

and here the sub-folders appear, we got two of them, data and code.
Next, let's see how our data looks like by listing a sample of it (e.g. 20 first rows)

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/first_twenty_rows.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/first_twenty_rows.jpg "Figure3. First twenty rows of purshases.txt")

Now, let's move our dataset (purshases.txt) to the HDFS and put it inside a directory called inputdata to have it ready to be processed.

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/move_data_to_hdfs.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/move_data_to_hdfs.jpg "Figure4. Move dataset to HDFS")

## Solution logic
Since mapReduce output job returns always the result in key-value format, so when we want to find the total sales per store, we're waiting for an output that's something like

<table style="margin: auto;" >
<tr><th>Store Location</th><th>Total Sales</th></tr>
<tr><td>Sydney</td><td>$1008523.00</td></tr>
<tr><td>NYC</td><td>$2214459.00</td></tr>
<tr><td>London</td><td>$7510044.00</td></tr>
<tr><td>...</td><td>...</td></tr>
</table><br>

So for this, we'll write a Mapper code that returns data in a key-value format, where it will list the entire sales but with only the store location and the amount columns.

and then, the Reducer comes after the shuffle and sort step, where it does the sum of the sales for each store location.

## Mapper Code
for the fun part, we'll have our code written in Python.

**Python**

{% highlight python %}
#!/usr/bin/python

# Format of each line is:
# date\ttime\tstore name\titem description\tcost\tmethod of payment
#
# We want elements 2 (store name) and 4 (cost)
# We need to write them out to standard output, separated by a tab

import sys

for line in sys.stdin:
    data = line.strip().split("\t")
    if len(data) == 6:
        date, time, store, item, cost, payment = data
        print "{0}\t{1}".format(store, cost)

{% endhighlight %}

It's so important to test the code before running the complete MapReduce job and applying it to the entire dataset.
One of the cool features in Hadoop streaming is that we can test out mapper.py code just by executing the script, entering some sample data and hitting Ctrl+d to simulate the output. Here's how it looks

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/test_manually.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/test_manually.jpg "Figure5. Testing Mapper code Manually")

Even better, we can just build a small set data file and pipe that into the mapper, so let's trying this

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/test_mapper_with_small_data.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/test_mapper_with_small_data.jpg "Figure6. Test Mapper With Small Data Locally")

As you can see, it's by accessing Hadoop's an efficient way to test the code rather than running the entire Hadoop job each time we edit the code!

PS: You can find the code on Github [here](https://github.com/ilyeSudo/mapReduce_blog_post)

## Reducer Code
As same as the mapper part, we will see how the reducer code is written in Python language

**Python**
{% highlight python %}
#!/usr/bin/python

import sys

salesTotal = 0
oldKey = None

# Loop around the data
# It will be in the format key\tval
# Where key is the store name, val is the sale amount
#
# All the sales for a particular store will be presented,
# then the key will change and we'll be dealing with the next store

for line in sys.stdin:
    data_mapped = line.strip().split("\t")
    if len(data_mapped) != 2:
        # Something has gone wrong. Skip this line.
        continue

    thisKey, thisSale = data_mapped

    if oldKey and oldKey != thisKey:
        print oldKey, "\t", salesTotal
        oldKey = thisKey;
        salesTotal = 0

    oldKey = thisKey
    salesTotal += float(thisSale)

if oldKey != None:
    print oldKey, "\t", salesTotal

{% endhighlight %}

As we tested the mapper code, we can do exactly the same thing with the reducer code.

PS: You can find the code on Github [here](https://github.com/ilyeSudo/mapReduce_blog_post)

## Test Locally
We are ready to run our entire test locally. The command to do so from inside the code folder is

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/test_mapper_and_reducer_with_small_data.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/test_mapper_and_reducer_with_small_data.jpg "Figure7. Test Mapper and Reducer code With Small Data Locally")

Let's break this down into pieces.
We use cat to access the entire content of test_in.txt. The | command says to take the standard output of the command on the left (cat) and feed it as standard input to the command on the right. This way the mapper script receives as standard input the contents of our file and it can read it line by line.

We then pass the output of our mapper (lines of key-value pairs) to the sort utility. This will sort the output so that all rows with the same key
are grouped together. Note that when running a Hadoop job, we don’t need to worry about sorting our results, because Hadoop does that for us (during the shuffle-sort phase). Here we need to include it since we
are testing our mapper and reducer locally.

We then input the sorted key-value pairs into the reducer. It then prints (as standard output, on the terminal) the final reduced output. Alternatively, we can save it to a file by appending the >> test_out.txt command at the end.

Great! We have obtained some reasonable results based on our small sample of 100 records. We can now proceed to run the job on Hadoop with the complete data set with more confidence.

## Running Hadoop job
Here the fun part comes, while having our code ready to go, we launch the Hadoop job simply by typing this long command

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/entire_hadoop_job.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/entire_hadoop_job.jpg "Figure8. Run The Entire Hadoop Job")

While the command being executed, we could follow the process by acceessing hadoop's web application running in the localhost **http://localhost:50050/jobtracker.jsp**

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/gui_localhost.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/gui_localhost.jpg "Figure9. Hadoop Web App GUI")

As the Figure8 shows, the job's output is stored in joboutput, we access it by typing ```$ hadoop fs -cat joboutput/part-00000
```

[<img src="/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/displaying_joboutput.jpg" style="height: 80%;width: 80%;"/>](/assets/img/blog/Hadoop-For-Dummies-Part1-Mapreduce/displaying_joboutput.jpg "Figure10. Displaying Job Output")

So here we come to the end of this post, hope you had a clearer idea how MapReduce job workes.
In the next blog posts, we'll walk through the other technologies of Hadoop's ecosystem.

PS: You can find the code on Github [here](https://github.com/ilyeSudo/mapReduce_blog_post)
