# Performance test report for the Cards Against Humanity Node.js API

## Tooling
The selected tools for this analysis are JMeter, Loader.io and WRK.

### JMeter
The [Apache JMeter™](https://jmeter.apache.org/) application is open source software, a 100% pure Java application designed to load test functional behavior and measure performance. It was originally designed for testing Web Applications but has since expanded to other test functions.

### Loader.io
[Loader.io](https://loader.io/) is a FREE load testing service that allows you to stress test your web-apps & apis with thousands of concurrent connections.

### WRK
[wrk](https://github.com/wg/wrk) is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU.

## Pre-requisites
The above mentioned tooling needs to be downloaded, as a project pre-requisite. Please refer to each tooling website for more information on how to install it. For Loader.io, a web account is required and the testing can be all performed online, without any software download.

## Objectives
The testing objective is to initially establish a performance a and load baseline for the [Cards Against Humanity Node.JS application](https://github.com/fartec0/cah-node-api/), currently forked in this repository.

### Performance and load baseline
Load spike: By measuring the performance metrics while creating a high quantity of connections (e.g. 10,000), during a certain period of time (e.g 15s) the application response time and latency distribution are registered. Thus providing a load/stress baseline for traffic spikes. 
Besides measuring the network metrics, the hardware metrics can be also monitored.

### Defining an acceptable response time
A industry-wide definition for an acceptable (web page) response time is based on [Nielsen's](https://www.nngroup.com/articles/response-times-3-important-limits/) definition:
> Jakob Nielsen defined the 3 response-time limits which are determined by human perceptual abilities:
0.1 seconds. This limit gives users the feeling of instantaneous response. This level of responsiveness is essential to support the feeling of direct manipulation. It’s also an ideal response time for the website.
1 second. One second keeps the user’s flow almost seamless. While users notice a slight delay, they still feel in control of the experience.
10 seconds is the limit for the user’s attention. For delays of more than 10 seconds, users will want to perform other tasks while waiting for the computer to finish. A 10-second delay in the web without any feedback will often make visitors leave a site immediately.

But regarding an API application, the response time is in general within the milliseconds range, and not in seconds as the definiton applicable for web pages performance. In this case, it is important to first establish a project-wide performance baseline and keep monitoring it during the project lifetime. 
For the modern web, any latency reduction is [very valuable](https://www.gigaspaces.com/blog/amazon-found-every-100ms-of-latency-cost-them-1-in-sales) and should be a focus within the software project.

In order to establish a performance baseline, measurements can be performed, using a customer-realistic load. During the sofware development cycle, performance tests can be executed and the initial baseline used for comparison. Also it is important that no socket errors are encountered during the baseline definition.

For example, the following performance baseline was gathered using WRK tool, considering the below scenario a customer-realistics load:
- Endpoint: https://caha-api-test.herokuapp.com/pick
- Test duration: 15.02s
- Number of open connections: 100
- Number of threads (users): 10
- Total number of requests: 6539 (read)
- Latency distribution: 119,45ms for the 99% percentile
- Socket errors: 0

``` bash
wrk --latency --timeout 3s --threads 10 --connections 100 --duration 15s https://caha-api-test.herokuapp.com/pick
Running 15s test @ https://caha-api-test.herokuapp.com/pick
  10 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    59.17ms   12.45ms 149.38ms   87.90%
    Req/Sec    44.30     19.98   101.00     64.98%
  Latency Distribution
     50%   57.33ms
     75%   63.01ms
     90%   71.25ms
     99%  119.45ms
  6539 requests in 15.02s, 4.71MB read
  Socket errors: connect 0, read 6539, write 0, timeout 0
  Non-2xx or 3xx responses: 6539
Requests/sec:    435.46
Transfer/sec:    321.02KB
```


#### Hardware metrics
For this project, the selected tool was [PM2](https://pm2.io), as it can provide realtime hardware metrics for the Node.JS application.

Example from PM2 dashboard, monitoring the `/fartec0/cah-node-api` Heroku application:
![PM2-Dashboard](https://user-images.githubusercontent.com/1813225/156917326-f64504cc-f3cb-4e75-ade8-c1965b16cd00.png)

### Load spike Baseline 
In this case we're looking for stabilishing load spike baseline metrics, and ultimately making sure that the applicatin is able to handle a very high amount of requests. 

Expected results:
- The application should not stop responding, or show signs of Hardware resources exhaustion.
- There should be no socket errors during the testing period.
- The [Latency distribution](https://engineering.linkedin.com/performance/who-moved-my-99th-percentile-latency), majorly at 99% should be within a pre-determined acceptable response time and throughput.

#### WRK results showing socket errors
- Test duration: 15.10s
- Total connections: 10,000
- Total number of requests: 53374
- Latency distribution: 125.85ms for the 99% percentile

``` bash
Running 15s test @ https://caha-api-test.herokuapp.com/
  10 threads and 10000 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency    64.32ms   22.39ms 502.13ms   93.71%
    Req/Sec   522.92    355.07     1.44k    54.20%
  Latency Distribution
     50%   60.81ms
     75%   67.61ms
     90%   78.78ms
     99%  125.85ms
  53374 requests in 15.10s, 17.81MB read
  Socket errors: connect 9761, read 0, write 0, timeout 0
Requests/sec:   3535.17
Transfer/sec:      1.18MB
```
