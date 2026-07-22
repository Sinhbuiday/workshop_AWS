---
title: "Blog 3"
date: "2023-07-17"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
---

# Capture clickstream data using AWS serverless services
<span class="meta-info">by Pritam Bedse and Jared Warren | on 17 JUL 2023 | in</span> [Advanced (300)](https://aws.amazon.com/blogs/industries/category/cross-industry/), [AWS for Industries](https://aws.amazon.com/blogs/industries/) | [Permalink](https://aws.amazon.com/blogs/industries/capture-clickstream-data-using-aws-serverless-services/)

Clickstream data refers to the collection of digital interactions that occur between a user and a website or mobile application. Capturing and creating usable insights from user data in real-time can be challenging. 

This post details how to leverage Amazon Web Services (AWS) serverless services to build a scalable architecture to seamlessly capture, process, visualize, and load clickstream data into analytics platforms.

---

## Solution Architecture

The solution uses Amazon API Gateway, AWS Lambda, and Amazon Kinesis Data Streams to ingest and process clickstream data. It then uses Amazon Kinesis Data Firehose to save the raw data in Amazon S3, followed by Amazon Athena and Amazon QuickSight to analyze and visualize user interactions.

![Clickstream Architecture Diagram](/images/3-Blogs/Blog-3/image-1.png)
<span class="meta-info">*Figure 1 — Serverless Clickstream Data Flow Architecture*</span>

1. **Ingestion:** The client (e.g., customer web portal) sends the clickstream payload (records of clicks, page views, duration, form submissions) to **Amazon API Gateway**.
2. **Standardization:** API Gateway transmits the record to **AWS Lambda**, where the data is standardized and cleaned.
3. **Streaming:** Lambda sends the standardized records to **Amazon Kinesis Data Streams** for asynchronous, real-time streaming at scale.
4. **Buffering & Delivery:** Kinesis Data Streams transfers the data to **Amazon Kinesis Data Firehose**, which buffers the records and uploads them to an **Amazon S3** bucket.
5. **Analytics:** **Amazon Athena** runs SQL queries directly against the data stored in S3.
6. **Visualization:** **Amazon QuickSight** connects to Athena to display modern, interactive BI dashboards.

---

## Why Choose Serverless for Clickstream?

* **Highly Variable Workloads:** Clickstream traffic volume naturally fluctuates depending on user activity. Serverless services automatically scale to handle traffic spikes (such as annual enrollment or sales periods) without manual capacity planning.
* **Cost-Efficient (Pay-as-you-go):** With no server maintenance overhead, you only pay for the execution time and data throughput consumed, preventing overprovisioning.
* **Real-time Insights:** Streamlining ingestion via Kinesis and Lambda ensures near real-time ingestion, allowing businesses to immediately evaluate new layout performances or marketing campaigns.

---

## Testing the Pipeline

To validate the architecture, a data generator Lambda function simulates real-time user activity. The payload consists of key fields representing typical e-commerce interactions:
* `customerid` / `deviceid`: Unique identifiers for tracking user profiles.
* `productid` / `productcategory` / `productsubcategory`: Product details explored by the user.
* `activitytype`: The specific action taken (e.g., product click, add to cart, checkout).

Once generated, records are buffered by Kinesis Data Firehose for 60 seconds (the minimum buffer interval) before being delivered to Amazon S3 as CSV files. Users can immediately preview and query the table using standard SQL in the **Amazon Athena** query editor.

---

## Conclusion
Leveraging AWS serverless services provides a powerful, highly scalable, and secure solution for capturing clickstream data. It enables data-driven decision-making and enhances customer experiences while eliminating the operational burden of managing complex server infrastructure.

For more details, see the original post on [AWS for Industries Blog: Capture clickstream data using AWS serverless services](https://aws.amazon.com/blogs/industries/capture-clickstream-data-using-aws-serverless-services/).