+++
author = "Jose Torrado"
title = "Anatomy of a Streaming Pipeline"
date = "2025-01-26"
description = ""
tags = [
    "data-engineering",
    "pipelines"
]
categories = [
    "architecture",
    "streaming",
]
image = "streaming-pipeline-cover.png"
+++

This is going to be a long one—mainly because it covers a *very* popular topic in the Data Engineering realm. Enjoy :)

---

### What Is a Streaming Pipeline?

A streaming pipeline is typically defined as a system that processes data in real time. In other words, as soon as new records are generated, they’re fed into and processed by your pipeline. There’s little to no waiting around—everything happens (almost) immediately.

A useful distinction can be made between true streaming and *near* real-time processing:

- **Streaming**: Each record is processed continuously the moment it arrives.  
- **Near Real-Time**: Small batches are processed at very short intervals. Exactly *how* small these batches should be is somewhat subjective; in many contexts, anything under an hour is still considered near real-time.

---

### How Does a Streaming Pipeline Differ from a Batch Pipeline?

A **batch pipeline** behaves like a sequence of tasks (often orchestrated in DAGs—[see my Airflow article](https://torrado.io/p/how-to-get-started-with-airflow-for-data-engineering/)) that run on a schedule. In each run, the batch job processes a specific time range of data.

A **streaming pipeline**, on the other hand, behaves more like a server: it’s continuously running, processing new data as it arrives. Because it’s always on, uptime matters *a lot*. Any interruption directly affects how quickly data gets processed and can lead to downstream backlog issues.

---

## Core Anatomy of a Streaming Pipeline

Below is a general breakdown of components you’ll find in many streaming pipelines:

1. **High-Frequency Data Sources**  
   Using a streaming pipeline only makes sense for the right use case—one that produces data at a high rate or that genuinely needs immediate processing. Think IoT sensors sending temperature data every second, or thousands of website clicks firing off events in real time.

2. **Messaging Queues**  
   These incoming data points are often fed into a queue or broker like Kafka, RabbitMQ, or Pub/Sub. This layer helps manage record throughput and can organize their order based on timestamp or other fields. It also buffers and distributes records to downstream components at a stable rate.

3. **Compute/Processing Layer**  
   This is where the data is enriched, aggregated, or transformed. Common frameworks include [Apache Flink](https://flink.apache.org/) and Spark Structured Streaming. For ultra-low latency needs (e.g., high-frequency trading), you might even see streaming applications built in C++ (most likely) or Java (meh). The choice really depends on your business requirements and latency constraints.

4. **Data Sink**  
   Once processed, data lands in its “final” destination: perhaps another Kafka topic, a real-time dashboard, or a relational database like Postgres. Sometimes, you need multiple sinks—one for real-time analytics and another for archival or further downstream processing.

5. **Metrics and Observability**  
   Streaming applications need robust logging and monitoring. In many cases, you’ll want a parallel process that tracks metrics (e.g., throughput, error rates) and writes them to some reliable sink. This ensures you can debug issues, maintain uptime, and scale effectively.

---

### Challenges of a Streaming Pipeline

While streaming pipelines are powerful, they bring unique complexities:

1. **Input Timeliness & Latency**  
   Data can arrive out of order, or certain events might trickle in slower than expected. If your pipeline doesn’t handle these edge cases, you could end up with incorrect results. Imagine executing a buy order based on a stock price that is hours behind the current value.

2. **Application Downtime**  
   If your streaming app goes down, unprocessed data piles up in the queue. In Kafka-speak, this increases your “consumer lag.” Restarting the app means it has to catch up, potentially processing large volumes of backlogged data. Hence, continuous uptime and thorough testing become critical.

Luckily, modern frameworks like Flink offer built-in mechanisms for stateful stream processing, handling a lot of the complexities around event ordering and time windows (to an extent).

---

### When Does It Make Sense to Use a Streaming Pipeline?

The **main benefit** of a streaming pipeline is low latency. If you truly need real-time insights—whether for triggering immediate actions or providing up-to-the-minute analytics—streaming pipelines shine.

- **High-Frequency Market Trading**: Milliseconds can determine profit or loss.  
- **Real-Time User Engagement**: Websites or apps that personalize content instantly.  
- **IoT and Sensor Data**: Rapid responses to environmental changes.

That said, streaming can be a costly and complex mistake if you don’t genuinely need it. Many stakeholders *think* they want real-time data when, in reality, a consistent 30-minute or hourly refresh is enough. Maintaining a streaming pipeline is significantly more expensive than running a daily batch job—so always consider the true business value before going all-in on real-time.

In my experience, most requests for “real-time” data end up being more about reliability and predictable refresh intervals. So before you spin up a Flink cluster, ask: “Would a batch job be enough?”

---

That’s a wrap for now. Streaming pipelines can be a powerful part of your data engineering toolbox—just make sure you need *that* level of power before you roll it out. 

:v:

