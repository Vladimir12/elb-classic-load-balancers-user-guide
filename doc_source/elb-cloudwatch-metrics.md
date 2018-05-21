# CloudWatch Metrics for Your Classic Load Balancer<a name="elb-cloudwatch-metrics"></a>

Elastic Load Balancing publishes data points to Amazon CloudWatch for your load balancers and your back\-end instances\. CloudWatch enables you to retrieve statistics about those data points as an ordered set of time\-series data, known as *metrics*\. Think of a metric as a variable to monitor, and the data points as the values of that variable over time\. For example, you can monitor the total number of healthy EC2 instances for a load balancer over a specified time period\. Each data point has an associated time stamp and an optional unit of measurement\.

You can use metrics to verify that your system is performing as expected\. For example, you can create a CloudWatch alarm to monitor a specified metric and initiate an action \(such as sending a notification to an email address\) if the metric goes outside what you consider an acceptable range\.

Elastic Load Balancing reports metrics to CloudWatch only when requests are flowing through the load balancer\. If there are requests flowing through the load balancer, Elastic Load Balancing measures and sends its metrics in 60\-second intervals\. If there are no requests flowing through the load balancer or no data for a metric, the metric is not reported\.

For more information about Amazon CloudWatch, see the *[Amazon CloudWatch User Guide](http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/)*\.

**Topics**
+ [Classic Load Balancer Metrics](#loadbalancing-metrics-clb)
+ [Metric Dimensions for Classic Load Balancers](#load-balancer-metric-dimensions-clb)
+ [Statistics for Classic Load Balancer Metrics](#measure-stats)
+ [View CloudWatch Metrics for Your Load Balancer](#ViewingDataUsingCloudWatch)
+ [Create CloudWatch Alarms for Your Load Balancer](#create_cw_alarms)

## Classic Load Balancer Metrics<a name="loadbalancing-metrics-clb"></a>

The `AWS/ELB` namespace includes the following metrics\.


| Metric | Description | 
| --- | --- | 
| BackendConnectionErrors |  The number of connections that were not successfully established between the load balancer and the registered instances\. Because the load balancer retries the connection when there are errors, this count can exceed the request rate\. Note that this count also includes any connection errors related to health checks\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Average`, `Minimum`, and `Maximum` are reported per load balancer node and are not typically useful\. However, the difference between the minimum and maximum \(or peak to average or average to trough\) might be useful to determine whether a load balancer node is an outlier\. **Example**: Suppose that your load balancer has 2 instances in us\-west\-2a and 2 instances in us\-west\-2b, and that attempts to connect to 1 instance in us\-west\-2a result in back\-end connection errors\. The sum for us\-west\-2a includes these connection errors, while the sum for us\-west\-2b does not include them\. Therefore, the sum for the load balancer equals the sum for us\-west\-2a\.  | 
| HealthyHostCount |  The number of healthy instances registered with your load balancer\. A newly registered instance is considered healthy after it passes the first health check\. If cross\-zone load balancing is enabled, the number of healthy instances for the `LoadBalancerName` dimension is calculated across all Availability Zones\. Otherwise, it is calculated per Availability Zone\. **Reporting criteria**: There are registered instances **Statistics**: The most useful statistics are `Average` and `Maximum`\. These statistics are determined by the load balancer nodes\. Note that some load balancer nodes might determine that an instance is unhealthy for a brief period while other nodes determine that it is healthy\. **Example**: Suppose that your load balancer has 2 instances in us\-west\-2a and 2 instances in us\-west\-2b, us\-west\-2a has 1 unhealthy instance, and us\-west\-2b has no unhealthy instances\. With the `AvailabilityZone` dimension, there is an average of 1 healthy and 1 unhealthy instance in us\-west\-2a, and an average of 2 healthy and 0 unhealthy instances in us\-west\-2b\.  | 
|  `HTTPCode_Backend_2XX`, `HTTPCode_Backend_3XX`, `HTTPCode_Backend_4XX`, `HTTPCode_Backend_5XX`  |  \[HTTP listener\] The number of HTTP response codes generated by registered instances\. This count does not include any response codes generated by the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` are all 1\. **Example**: Suppose that your load balancer has 2 instances in us\-west\-2a and 2 instances in us\-west\-2b, and that requests sent to 1 instance in us\-west\-2a result in HTTP 500 responses\. The sum for us\-west\-2a includes these error responses, while the sum for us\-west\-2b does not include them\. Therefore, the sum for the load balancer equals the sum for us\-west\-2a\.  | 
| HTTPCode\_ELB\_4XX |  \[HTTP listener\] The number of HTTP 4XX client error codes generated by the load balancer\. Client errors are generated when a request is malformed or incomplete\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` are all 1\. **Example**: Suppose that your load balancer has us\-west\-2a and us\-west\-2b enabled, and that client requests include a malformed request URL\. As a result, client errors would likely increase in all Availability Zones\. The sum for the load balancer is the sum of the values for the Availability Zones\.  | 
| HTTPCode\_ELB\_5XX |  \[HTTP listener\] The number of HTTP 5XX server error codes generated by the load balancer\. This count does not include any response codes generated by the registered instances\. The metric is reported if there are no healthy instances registered to the load balancer, or if the request rate exceeds the capacity of the instances \(spillover\) or the load balancer\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` are all 1\. **Example**: Suppose that your load balancer has us\-west\-2a and us\-west\-2b enabled, and that instances in us\-west\-2a are experiencing high latency and are slow to respond to requests\. As a result, the surge queue for the load balancer nodes in us\-west\-2a fills and clients receive a 503 error\. If us\-west\-2b continues to respond normally, the sum for the load balancer equals the sum for us\-west\-2a\.  | 
| Latency |  \[HTTP listener\] The total time elapsed, in seconds, from the time the load balancer sent the request to a registered instance until the instance started to send the response headers\. \[TCP listener\] The total time elapsed, in seconds, for the load balancer to successfully establish a connection to a registered instance\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Average`\. Use `Maximum` to determine whether some requests are taking substantially longer than the average\. Note that `Minimum` is typically not useful\. **Example**: Suppose that your load balancer has 2 instances in us\-west\-2a and 2 instances in us\-west\-2b, and that requests sent to 1 instance in us\-west\-2a have a higher latency\. The average for us\-west\-2a has a higher value than the average for us\-west\-2b\.  | 
| RequestCount |  The number of requests completed or connections made during the specified interval \(1 or 5 minutes\)\. \[HTTP listener\] The number of requests received and routed, including HTTP error responses from the registered instances\. \[TCP listener\] The number of connections made to the registered instances\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Minimum`, `Maximum`, and `Average` all return 1\. **Example**: Suppose that your load balancer has 2 instances in us\-west\-2a and 2 instances in us\-west\-2b, and that 100 requests are sent to the load balancer\. There are 60 requests sent to us\-west\-2a, with each instance receiving 30 requests, and 40 requests sent to us\-west\-2b, with each instance receiving 20 requests\. With the `AvailabilityZone` dimension, there is a sum of 60 requests in us\-west\-2a and 40 requests in us\-west\-2b\. With the `LoadBalancerName` dimension, there is a sum of 100 requests\.  | 
| SpilloverCount |  The total number of requests that were rejected because the surge queue is full\. \[HTTP listener\] The load balancer returns an HTTP 503 error code\. \[TCP listener\] The load balancer closes the connection\. **Reporting criteria**: There is a nonzero value **Statistics**: The most useful statistic is `Sum`\. Note that `Average`, `Minimum`, and `Maximum` are reported per load balancer node and are not typically useful\. **Example**: Suppose that your load balancer has us\-west\-2a and us\-west\-2b enabled, and that instances in us\-west\-2a are experiencing high latency and are slow to respond to requests\. As a result, the surge queue for the load balancer node in us\-west\-2a fills, resulting in spillover\. If us\-west\-2b continues to respond normally, the sum for the load balancer will be the same as the sum for us\-west\-2a\.  | 
| SurgeQueueLength |  The total number of requests that are pending routing\. The load balancer queues a request if it is unable to establish a connection with a healthy instance in order to route the request\. The maximum size of the queue is 1,024\. Additional requests are rejected when the queue is full\. For more information, see `SpilloverCount`\. **Reporting criteria**: There is a nonzero value\.  **Statistics**: The most useful statistic is `Maximum`, because it represents the peak of queued requests\. The `Average` statistic can be useful in combination with `Minimum` and `Maximum` to determine the range of queued requests\. Note that `Sum` is not useful\. **Example**: Suppose that your load balancer has us\-west\-2a and us\-west\-2b enabled, and that instances in us\-west\-2a are experiencing high latency and are slow to respond to requests\. As a result, the surge queue for the load balancer nodes in us\-west\-2a fills, with clients likely experiencing increased response times\. If this continues, the load balancer will likely have spillovers \(see the `SpilloverCount` metric\)\. If us\-west\-2b continues to respond normally, the `max` for the load balancer will be the same as the `max` for us\-west\-2a\.  | 
| UnHealthyHostCount |  The number of unhealthy instances registered with your load balancer\. An instance is considered unhealthy after it exceeds the unhealthy threshold configured for health checks\. An unhealthy instance is considered healthy again after it meets the healthy threshold configured for health checks\. **Reporting criteria**: There are registered instances **Statistics**: The most useful statistics are `Average` and `Minimum`\. These statistics are determined by the load balancer nodes\. Note that some load balancer nodes might determine that an instance is unhealthy for a brief period while other nodes determine that it is healthy\. **Example**: See `HealthyHostCount`\.  | 

<a name="estimated-metrics"></a>The following metrics enable you to estimate your costs if you migrate a Classic Load Balancer to an Application Load Balancer\. These metrics are intended for informational use only, not for use with CloudWatch alarms\. Note that if your Classic Load Balancer has multiple listeners, these metrics are aggregated across the listeners\.

These estimates are based on a load balancer with one default rule and a certificate that is 2K in size\. If you use a certificate that is 4K or greater in size, we recommend that you estimate your costs as follows: create an Application Load Balancer based on your Classic Load Balancer using the migration tool and monitor the `ConsumedLCUs` metric for the Application Load Balancer\. For more information, see [Migrate from a Classic Load Balancer to an Application Load Balancer](http://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/migrate-to-application-load-balancer.html) in the *Elastic Load Balancing User Guide*\.


| Metric | Description | 
| --- | --- | 
| EstimatedALBActiveConnectionCount |  The estimated number of concurrent TCP connections active from clients to the load balancer and from the load balancer to targets\.  | 
| EstimatedALBConsumedLCUs |  The estimated number of load balancer capacity units \(LCU\) used by an Application Load Balancer\. You pay for the number of LCUs that you use per hour\. For more information, see [Elastic Load Balancing Pricing](https://aws.amazon.com/elasticloadbalancing/pricing/)\.  | 
| EstimatedALBNewConnectionCount |  The estimated number of new TCP connections established from clients to the load balancer and from the load balancer to targets\.  | 
| EstimatedProcessedBytes |  The estimated number of bytes processed by an Application Load Balancer\.  | 

## Metric Dimensions for Classic Load Balancers<a name="load-balancer-metric-dimensions-clb"></a>

To filter the metrics for your Classic Load Balancer, use the following dimensions\.


|  Dimension  |  Description  | 
| --- | --- | 
|  AvailabilityZone  |  Filter the metric data by the specified Availability Zone\.  | 
|  LoadBalancerName  |  Filter the metric data by the specified load balancer\.  | 

## Statistics for Classic Load Balancer Metrics<a name="measure-stats"></a>

CloudWatch provides statistics based on the metric data points published by Elastic Load Balancing\. Statistics are metric data aggregations over specified period of time\. When you request statistics, the returned data stream is identified by the metric name and dimension\. A dimension is a name/value pair that uniquely identifies a metric\. For example, you can request statistics for all the healthy EC2 instances behind a load balancer launched in a specific Availability Zone\.

The `Minimum` and `Maximum` statistics reflect the minimum and maximum reported by the individual load balancer nodes\. For example, suppose there are 2 load balancer nodes\. One node has `HealthyHostCount` with a `Minimum` of 2, a `Maximum` of 10, and an `Average` of 6, while the other node has `HealthyHostCount` with a `Minimum` of 1, a `Maximum` of 5, and an `Average` of 3\. Therefore, the load balancer has a `Minimum` of 1, a `Maximum` of 10, and an `Average` of about 4\.

The `Sum` statistic is the aggregate value across all load balancer nodes\. Because metrics include multiple reports per period, `Sum` is only applicable to metrics that are aggregated across all load balancer nodes, such as `RequestCount`, `HTTPCode_ELB_XXX`, `HTTPCode_Backend_XXX`, `BackendConnectionErrors`, and `SpilloverCount`\.

The `SampleCount` statistic is the number of samples measured\. Because metrics are gathered based on sampling intervals and events, this statistic is typically not useful\. For example, with `HealthyHostCount`, `SampleCount` is based on the number of samples that each load balancer node reports, not the number of healthy hosts\.

A percentile indicates the relative standing of a value in a data set\. You can specify any percentile, using up to two decimal places \(for example, p95\.45\)\. For example, the 95th percentile means that 95 percent of the data is below this value and 5 percent is above\. Percentiles are often used to isolate anomalies\. For example, suppose that an application serves the majority of requests from a cache in 1\-2 ms, but in 100\-200 ms if the cache is empty\. The maximum reflects the slowest case, around 200 ms\. The average doesn't indicate the distribution of the data\. Percentiles provide a more meaningful view of the application's performance\. By using the 99th percentile as an Auto Scaling trigger or a CloudWatch alarm, you can target that no more than 1 percent of requests take longer than 2 ms to process\.

## View CloudWatch Metrics for Your Load Balancer<a name="ViewingDataUsingCloudWatch"></a>

You can view the CloudWatch metrics for your load balancers using the Amazon EC2 console\. These metrics are displayed as monitoring graphs\. The monitoring graphs show data points if the load balancer is active and receiving requests\.

Alternatively, you can view metrics for your load balancer using the CloudWatch console\.

**To view metrics using the Amazon EC2 console**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select your load balancer\.

1. Choose the **Monitoring** tab\.

1. \(Optional\) To filter the results by time, select a time range from **Showing data for**\.

1. To get a larger view of a single metric, select its graph\. The following metrics are available:
   + Healthy Hosts — `HealthyHostCount`
   + Unhealthy Hosts — `UnHealthyHostCount`
   + Average Latency — `Latency`
   + Sum Requests — `RequestCount`
   + Backend Connection Errors — `BackendConnectionErrors`
   + Surge Queue Length — `SurgeQueueLength`
   + Spillover Count — `SpilloverCount`
   + Sum HTTP 2XXs — `HTTPCode_Backend_2XX`
   + Sum HTTP 4XXs — `HTTPCode_Backend_4XX`
   + Sum HTTP 5XXs — `HTTPCode_Backend_5XX`
   + Sum ELB HTTP 4XXs — `HTTPCode_ELB_4XX`
   + Sum ELB HTTP 5XXs — `HTTPCode_ELB_5XX`

**To view metrics using the CloudWatch console**

1. Open the CloudWatch console at [https://console\.aws\.amazon\.com/cloudwatch/](https://console.aws.amazon.com/cloudwatch/)\.

1. In the navigation pane, choose **Metrics**\.

1. Select the **ELB** namespace\.

1. Do one of the following:
   + Select a metric dimension to view metrics by load balancer, by Availability Zone, or across all load balancers\.
   + To view a metric across all dimensions, type its name in the search field\.
   + To view the metrics for a single load balancer, type its name in the search field\.
   + To view the metrics for a single Availability Zone, type its name in the search field\.

## Create CloudWatch Alarms for Your Load Balancer<a name="create_cw_alarms"></a>

An alarm watches a single metric over the time period that you specify\. Depending on the value of the metric relative to a threshold that you define, the alarm can send one or more notifications using Amazon SNS, a service that enables applications, end users, and devices to instantly send and receive notifications\. For more information, see [Get Started with Amazon SNS](http://docs.aws.amazon.com/sns/latest/gsg/Welcome.html)\.

An alarm sends notifications to Amazon SNS when the specified metric reaches the defined range and remains in that range for a specified period of time\. An alarm has three possible states:
+ `OK`—The value of the metric is within the range you've specified\.
+ `ALARM`—The value of the metric is outside the range that you've specified for the specified period of time\.
+ `INSUFFICIENT_DATA`—Either the metric is not yet available or there is not enough data to determine the alarm state\.

Whenever the state of an alarm changes, CloudWatch uses Amazon SNS to send a notification to the email addresses that you specified\.

Use the following procedure to create an alarm for your load balancer using the Amazon EC2 console\. The alarm sends notifications to an SNS topic whenever the load balancer's latency is above 120 seconds for 1 consecutive period of 5 minutes\. Note that a short period creates a more sensitive alarm, while a longer period can mitigate brief spikes in a metric\.

**Note**  
Alternately, you can create an alarm for your load balancer using the CloudWatch console\. For more information, see [Send Email Based on Load Balancer Alarm](http://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/US_AlarmAtThresholdELB.html) in the *Amazon CloudWatch User Guide*\.

**To create an alarm for your load balancer**

1. Open the Amazon EC2 console at [https://console\.aws\.amazon\.com/ec2/](https://console.aws.amazon.com/ec2/)\.

1. On the navigation pane, under **LOAD BALANCING**, choose **Load Balancers**\.

1. Select your load balancer\.

1. On the **Monitoring** tab, choose **Create Alarm**\.

1. If you have an SNS topic that you want to use, select it from **Send a notification to**\. Otherwise, create an SNS topic as follows:

   1. Choose **create topic**\.

   1. For **Send a notification to**, type a name for your topic\.

   1. For **With these recipients**, type the email addresses of the recipients to notify, separated by commas\. You can enter up to 10 email addresses\. Each recipient receives an email from Amazon SNS with a link to subscribe to the SNS topic in order to receive notifications\.

1. Define the threshold for your alarm as follows\. For **Whenever**, select **Average** and **Average Latency**\. For **Is**, select **>** and enter `120`\. For **For at least**, type `1` and select a consecutive period of **5 minutes**\.

1. For **Name of alarm**, a name is automatically generated for you\. If you prefer, you can type a different name\.

1. Choose **Create Alarm**\.