
# Quickstart for debugging PySpark applications

A sample application for debugging Python Spark apps using Rookout.

Before following this guide we recommend reading the basic [Python + Rookout](https://docs.rookout.com/docs/sdk-setup.html) guide.

## Running PySpark jobs with Rookout

1. *Clone and install dependencies*:
 ```bash
    git clone https://github.com/Rookout/tutorial-pyspark
    cd pyspark
    pip install -r requirements.txt  # also on executor nodes, if running in a cluster
```

2. *Export organization token and label*:
 ```
export ROOKOUT_TOKEN=<Your Rookout Token>
```
```
export ROOKOUT_LABELS=app:pyspark
```

3. *Try placing breakpoints at these locations in `example.py`*:
* In `main` on line 65
* In `map_partitions_handler` on line 48
* In `multiply_latlong` (a UDF) on line 24
* Run your program


4. *Use `spark-submit` to submit the job while loading Rookout into Spark executors (Spark standalone)*:
```
spark-submit --conf spark.python.daemon.module=rook.pyspark_daemon example.py
```


5. *Have fun debugging*:
	Go to https://app.rookout.com and start debugging :)

    #### Usage example
    You cannot place breakpoints in executor nodes in the `__main__` module, as a side effect of how PySpark serializes functions.

    Try placing breakpoints at:
    * In `example.py` in the main() method
    * In `example_executor_module.handle_record` on line 35
    * In `example_executor_module.multiply_latlong` (a UDF) on line 10
    * Run your program

## Running under YARN (AWS EMR)
1. Upload the `sample-data.csv` file to a S3 bucket, and modify the code so it calls `spark_session.read.csv` with `s3a://bucket-name/sample-data.csv` for the path, where `bucket-name` is replaced by the bucket that contains your uploaded file.
2. Specify the `ROOKOUT_TOKEN` environment variable:

```
spark-submit --conf spark.python.daemon.module=rook.pyspark_daemon --conf spark.yarn.appMasterEnv.ROOKOUT_TOKEN=[Your Rookout Token] --conf spark.executorEnv.ROOKOUT_TOKEN=[Your Rookout Token] example.py
```



## Rookout Integration explained
1. Rookout is loaded into the Spark driver on line 74 in main.py, by calling `rook.start()` directly. This is the standard way of loading Rookout, but we still haven't loaded Rookout into the executor nodes.

2. Specifying the configuration option `--conf spark.python.daemon.module=rook.pyspark_daemon` loads Rookout into the executor nodes. When specifying this option, Rookout will automatically load to worker processes. This is necessary for placing breakpoints in any code that runs on executor nodes.

## Limitations
Rookout cannot currently place executor breakpoints in nested functions, lambdas, staticmethods when those are used as a UDF or a mapPartitions callable, but you can place breakpoints if them if they are called by the UDF or the mapPartitions callable (as in the example).
In addition, you cannot place breakpoints in functions defined in the `__main__` module.
This is a side effect of how PySpark serializes functions before sending them to executors. 

[Python + Rookout]: https://docs.rookout.com/docs/sdk-setup.html
## Debugging Spark Engine Code
Rookout can be used to debug the Spark Engine code itself, such as a Spark Driver for example. Rookout can optionally be configured to debug Spark Java classes. To do that, set the `spark.driver.extraJavaOptions` config option allowing us to pass additional JVM options upon startup. We'll use this to pass the Rookout Java agent:
```
spark-submit --conf spark.python.daemon.mod ule=rook.pyspark_daemon --conf spark.driver.extraJavaOptions=-javaagent:rook.jar example.py
```
You can find additional details about the property here: https://spark.apache.org/docs/latest/configuration.html#dynamically-loading-spark-properties

You will also need to download the Rookout agent Jar file:
```
curl -L "https://repository.sonatype.org/service/local/artifact/maven/redirect?r=central-proxy&g=com.rookout&a=rook&v=LATEST" -o rook.jar
```
Apache Spark code can be found here: https://github.com/apache/spark
