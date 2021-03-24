# Using Spark with Polynote

Polynote has deep integration with [Apache Spark](https://spark.apache.org). However, it can also run without Spark support enabled.
In fact, since launching a Spark session can be expensive, Polynote runs your notebooks without Spark support by default.

Polynote attempts to be smart about whether to launch a notebook with Spark: it'll only launch a Spark session if you
have any Spark configuration parameters in the notebook config. So the fastest way to get started with Spark is to add
something like `spark.master: local[*]` to your notebook configuration, like so:

![spark-master-config](images/spark-master-config.png)