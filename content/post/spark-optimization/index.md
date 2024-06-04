+++
author = "Jose Torrado"
title = "Boost Your Pyspark Performance: Best Practices for Data Pipelines"
date = "2024-06-03"
description = "Make your pipelines lightning fast"
tags = [
    "spark",
    "optimization",
    "data-engineering"
]
categories = [
    "data-engineering",
    "pipelines"
]
image = "spark-optimization.png"
+++

# Optimizing for Spark 

Apache Spark is a powerful tool for big data processing, and PySpark makes it accessible to Python developers. However, to get the best performance out of your PySpark jobs, its a good idea to follow certain best practices.

I will be going over a few things to keep in mind when developing Pyspark code

## Use the DataFrame API instead of RDDs

>**Prefer using the DataFrame API over RDDs for data processing.**

Resilient Distributed Datasets (RDDs) do not provide optimization features like Catalyst optimizer and Tungsten execution engine, leading to inefficient execution plans and slower performance.

**How it Works:**
- Catalyst Optimizer: This is an extensible query optimizer that automatically optimizes the logical plan of operations. It applies various optimization rules such as predicate pushdown, constant folding, and projection pruning to minimize the amount of data processed and improve performance.

- Tungsten Execution Engine: Tungsten improves execution by optimizing memory and CPU usage. It uses off-heap memory for better cache management, and code generation to produce optimized bytecode for execution.

**Example:**

```Python
# Using DataFrame API
df = spark.read.csv("data.csv")
filtered_df = df.filter(df['age'] > 30)

# Instead of RDD
rdd = sc.textFile("data.csv")
filtered_rdd = rdd.filter(lambda row: int(row.split(",")[1]) > 30)
```

## Cache Intermediate Results

>**Cache intermediate DataFrames that are reused multiple times in your pipeline.**

This is particularly useful for smaller datasets that are reused multiple times in the code, for uses like mapping (to give an example). Because of Spark's [lazy evaluation](https://spark.apache.org/docs/latest/rdd-programming-guide#:~:text=All%20transformations%20in%20Spark%20are%20lazy%2C%20in%20that%20they%20do%20not%20compute%20their%20results%20right%20away.), any DataFrames that is to be used multiple times but is not cached will essentially be generated again - adding unecessary processing to your code.

Keep in mind that this data must be stored somewhere though, hence my emphasis on _small_ datasets. Spark does, however, provide several storage levels (e.g., MEMORY_ONLY, MEMORY_AND_DISK) to control how and where data is cached.

**Example:**

```python
# Cache the intermediate DataFrame
filtered_df.cache()
result = filtered_df.groupBy("age").count()
```

## Avoid Shuffling Data

>**Minimize data shuffling by using appropiate partitioning and avoiding wide transformations when possible.**

Remember Spark is a distributed computing framework - which means that data is randomly (let's assume random for the sake of this explanation) placed in different _executors_, or machines. Whenever an operation requires the same group of rows for an opperation, it needs to find it and move it to the correct machine.

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExYTA0ZXd6cGpjNjNxbDZxaWZ4dHExcGM0aTlybDM5NGdubHdwb2YzYyZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/QBpkGOjpn3NfVE5tZ4/giphy.gif)

Shuffling data across the network is expensive and can lead to performance bottlenecks. Shuffling occurs during operations like `groupBy`, `reduceByKey`, and `join`, which require data to be redistributed across different nodes.

By using narrow transformation (like `map`, `filter`) and appropiate partitioning (e.g. `partitioning`, `coalesce`), you can reduce the need for shuffling. Narrow transformations operate on a single partition, whereas wide transformations require data from multiple partitions.

**Example:**

```python
# Repartition the DataFrame to reduce shuffling
df = df.repartition("age")
result = df.groupBy("age").count()
```

## Use Broadcast Variables for Small Data

>**Use broadcast variables to efficiently join small datasets with large ones.**

Large joins can cause significant shuffling and slow down processing. This happens because each node needs to exchange data with all other nodes, leading to network congestion.

Broadcast variables allow you to send a read-only copy of a small dataset to all worker nodes. This eliminates the need for shuffling during joins, as each node can directly access the broadcasted data.

This will only help with small datasets though - broadcasting larger amounts of data can lead to dead executors and network congestion.

**Example:**

```pyhton
# Broadcast the small DataFrame
small_df = spark.read.csv("small_data.csv")
broadcast_small_df = sc.broadcast(small_df.collect()) # Where sc is your SparkContext

# Use the broadcast variable in a join
result = df.filter(df['id'].isin([row['id'] for row in broadcast_small_df.value]))
```

## Optimize Spark Configurations

>**Tune Spark configuration parameters to match your workload and cluster resources.**

Default configuration settings may not be optimal for your specific workload, leading to inefficient resource utilization and slower performance.

This, in my experience, can be the most finnicky part of working with Spark. It can feel like endless tunning of knobs to get the configurations just right.

![](https://i.giphy.com/media/v1.Y2lkPTc5MGI3NjExMndleWlrbHFlOHFmOWJ0NnA4ZGhqdTlvZWh6OWZqMWVucWcyaGh3ZiZlcD12MV9pbnRlcm5hbF9naWZfYnlfaWQmY3Q9Zw/jsrlW09L9Xg2vOwYrB/giphy.gif)

But it does not need to be a grueling process as long as you understand the basics of what each config is set to do. 

By adjusting parameters such as executor memory, number of cores, and parallelism, you can better utilize your cluster resources and improve job performance.
  - **spark.executor.memory:** Amount of memory allocated for each executor.
  - **spark.executor.cores:** Number of CPU cores allocated for each executor.
  - **spark.sql.shuffle.partitions:** Number of partitions to use when shuffling data for joins or aggregations.

There are additional configurations for network timeouts, shuffles and a variety of different aspects. Documentation will be your best friend here for debugging, you can find it [here](https://spark.apache.org/docs/latest/configuration)

**Example:**

```python
# Set Spark configuration parameters
spark.conf.set("spark.executor.memory", "4g")
spark.conf.set("spark.executor.cores", "2")
spark.conf.set("spark.sql.shuffle.partitions", "200")
```

# To close...

There are many ways you can optimize your Spark code. Besides these, my general advice would be to always keep in mind how Pyspark will process the data when you are developing your code. By following these best practices, you can significantly improve the performance of your PySpark jobs and ensure that your data pipelines run efficiently.

Let me know if I missed any of your favorite optimization tips! If you found this post helpful, check out my other blog posts for more insights on data engineering best practices.
