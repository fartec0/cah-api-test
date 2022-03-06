# Performance test report for the Cards Against Humanity Node.js API

## Tooling
The selected tools for this analysis are Loader.io and WRK.

### Loader.io
[Loader.io](https://loader.io/) is a FREE load testing service that allows you to stress test your web-apps & apis with thousands of concurrent connections.

### WRK
[wrk](https://github.com/wg/wrk) is a modern HTTP benchmarking tool capable of generating significant load when run on a single multi-core CPU.

## Pre-requisites
The above mentioned tooling needs to be downloaded, as a project pre-requisite. Please refer to each tooling website for more information on how to install it. For Loader.io, a web account is required and the testing can be all performed online, without any software download.

## System Under Test (SUT)
The application under test is a fork from the Cards Against Humanity Node.JS, currently deployed to [Heroku](www.heroku.com) and available at: https://dashboard.heroku.com/apps/cah-api-test 

There are three available API endpoints, that were used during this project:
``` txt
/question - get a random white card
/answer - get a random black card
/pick - get a question and answer randomly chosen
```

## Scope
The scope of this analysis is the `/pick` endpoint only, serving as a basis for measuring other endpoints, as it would be the most complex endpoint in the API.

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

## Performance baseline

In order to establish a performance baseline, measurements should be performed using a `customer-realistic` load. During the sofware development cycle, performance tests can be executed and the initial baseline used for comparison. Also it is important that no socket errors are encountered during the baseline definition.

### Customer realistic baselines

#### Regular load
The following performance baseline (free of socket errors) was gathered using WRK tool, and its using the considered customer-realistic scenario for this project for a regular customer usage load event:
- Endpoint: https://caha-api-test.herokuapp.com/pick
- **Test duration: 15.10**
- Number of open connections: 10
- **Number of threads (users): 10**
- Total number of requests: 1326 (read)
- Latency distribution: 310,96ms for the 99% percentile
- Latency distribution: 128,78ms for the 90% percentile
- **Socket errors: 0**
- 
``` bash
Running 15s test @ https://cah-api-test.herokuapp.com/pick
  10 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   111.47ms   38.25ms 321.80ms   92.41%
    Req/Sec     9.29      2.42    20.00     88.29%
  Latency Distribution
     50%  101.24ms
     75%  105.11ms
     90%  128.78ms
     99%  310.96ms
  1326 requests in 15.10s, 367.38KB read
Requests/sec:     87.80
Transfer/sec:     24.32KB
```

#### High load
The following performance baseline (free of socket errors) was gathered using WRK tool, and its using the considered customer-realistic scenario for this project for a high load event:
- Endpoint: https://caha-api-test.herokuapp.com/pick
- **Test duration: 15.10**
- Number of open connections: 100
- **Number of threads (users): 10**
- Total number of requests: 15750 (read)
- Latency distribution: 494,32ms for the 99% percentile
- Latency distribution: 302,15ms for the 90% percentile
- **Socket errors: 0**

``` bash
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   188.38ms   79.74ms 594.82ms   77.08%
    Req/Sec   108.59     54.86   230.00     61.67%
  Latency Distribution
     50%  168.28ms
     75%  217.91ms
     90%  302.15ms
     99%  494.32ms
  15750 requests in 15.10s, 4.28MB read
Requests/sec:   1042.82
Transfer/sec:    290.41KB
```

### Load spike Baseline 
In this case we're looking for stabilishing load spike baseline metrics, and ultimately making sure that the applicatin is able to handle a very high amount of requests. 

Expected results:
- The application should not stop responding, or show signs of Hardware resources exhaustion.
- There should be no socket errors during the testing period.
- The [Latency distribution](https://engineering.linkedin.com/performance/who-moved-my-99th-percentile-latency), majorly at 99% should be within a pre-determined acceptable response time and throughput.

While stressing the application with a high number or connections, during the same time period and with the same quantity of threads, it is visible that the number of socket errors increase with the load - as seen below from the output of one of the test results. 

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

#### Hardware metrics
For this project, the selected tool was [PM2](https://pm2.io), as it can provide realtime hardware metrics for the Node.JS application.

Example from PM2 dashboard, monitoring the `/fartec0/cah-node-api` Heroku application:
![PM2-Dashboard](https://user-images.githubusercontent.com/1813225/156917326-f64504cc-f3cb-4e75-ade8-c1965b16cd00.png)

## Monitoring
For monitoring the application response time and throughput, the [Heroku metrics dashboard](https://dashboard.heroku.com/apps/cah-api-test/metrics/web) was utilised. 

Image below, an extract from the Heroku metrics dashboard, showing spikes in `response time` and `throughput`, including 5XX erros during peak load tests:
![herokuDashboard](https://user-images.githubusercontent.com/1813225/156922757-01e9c983-eb5f-4e9d-9dad-d3676ea8c1c4.png)

Note that the SUT could not handle an excessive amount of requests, demonstrating that the network load should be reduced gradually until no socket errors are triggered, and thus identifying the actual application limit.

## Test results for the /pick endpoint with high load, increase of open connections
The test scenarios used the following values, with the objective of gradually scaling the number of open connections, while keeping socket errors inexistent.

#### 100 connections
- Test duration: 15.10s
- Total connections: 100
- Total number of requests: 12181
- Latency distribution: 389.30ms for the 99% percentile
- Socket errors: 0
``` bash
Running 15s test @ https://cah-api-test.herokuapp.com/pick
  10 threads and 100 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   122.57ms   51.39ms   1.10s    93.17%
    Req/Sec    84.47     23.26   140.00     74.11%
  Latency Distribution
     50%  107.39ms
     75%  123.15ms
     90%  155.46ms
     99%  389.30ms
  12181 requests in 15.10s, 3.31MB read
Requests/sec:    806.85
Transfer/sec:    224.76KB
```

#### 200 connections
- Test duration: 15.10s
- Total connections: 200
- Total number of requests: 15750
- Latency distribution: 494.32ms for the 99% percentile
- Socket errors: 0

Results
``` bash
  10 threads and 200 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   188.38ms   79.74ms 594.82ms   77.08%
    Req/Sec   108.59     54.86   230.00     61.67%
  Latency Distribution
     50%  168.28ms
     75%  217.91ms
     90%  302.15ms
     99%  494.32ms
  15750 requests in 15.10s, 4.28MB read
Requests/sec:   1042.82
Transfer/sec:    290.41KB
```

#### 300 connections - Start of 'connect' socket errors
- Test duration: 15.10s
- Total connections: 300
- Total number of requests: 20522
- Latency distribution: 438.44ms for the 99% percentile
- Socket errors: 61

``` bash
  10 threads and 300 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   171.75ms   78.50ms   2.35s    88.89%
    Req/Sec   144.00     76.28   353.00     65.55%
  Latency Distribution
     50%  151.84ms
     75%  190.96ms
     90%  261.16ms
     99%  438.44ms
  20522 requests in 15.10s, 5.58MB read
  Socket errors: connect 61, read 0, write 0, timeout 0
Requests/sec:   1359.12
Transfer/sec:    378.74KB
```

#### 400 connections - Increased socket errors
- Test duration: 15.10s
- Total connections: 300
- Total number of requests: 17466
- Latency distribution: 611.69ms for the 99% percentile
- Socket errors: 161

``` bash
  10 threads and 400 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   200.73ms   94.04ms 976.23ms   84.43%
    Req/Sec   145.90    107.56   410.00     64.42%
  Latency Distribution
     50%  177.79ms
     75%  223.26ms
     90%  306.71ms
     99%  611.17ms
  17466 requests in 15.10s, 4.75MB read
  Socket errors: connect 161, read 0, write 0, timeout 0
Requests/sec:   1156.41
Transfer/sec:    322.17KB
```

### Conclusion for the high number of open connections, 10 threads
Observations:
- The number of socket connect errors start to appear when 300 open connections are defined.
- The latency distribution at 99% percentile already starts as high as 389.30ms.
- The number of threads is kept at 10 during the whole testing execution, simulating a scaling of open connections, still for the same quantity of users (threads).

## Test results for the /pick endpoint with regular network load, scaling number of users and open connections
The test scenarios used the following values, with the objective of gradually scaling the number of open connections, while keeping socket errors inexistent.

#### 10 connections - 10 threads
- Test duration: 15.10s
- Total connections: 10
- Total number of requests: 1326
- Latency distribution: 128.78ms for the 90% percentile
- Latency distribution: 310.96ms for the 99% percentile
- Socket errors: 0

``` bash
  10 threads and 10 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   111.47ms   38.25ms 321.80ms   92.41%
    Req/Sec     9.29      2.42    20.00     88.29%
  Latency Distribution
     50%  101.24ms
     75%  105.11ms
     90%  128.78ms
     99%  310.96ms
  1326 requests in 15.10s, 367.38KB read
Requests/sec:     87.80
Transfer/sec:     24.32KB
```

#### 20 connections - 20 threads
- Test duration: 15.10s
- Total connections: 20
- Total number of requests: 2700
- Latency distribution: 117.77ms for the 90% percentile
- Latency distribution: 304.69ms for the 99% percentile
- Socket errors: 0

``` bash
Running 15s test @ https://cah-api-test.herokuapp.com/pick
  20 threads and 20 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   108.94ms   30.72ms 315.23ms   95.22%
    Req/Sec     9.32      2.17    20.00     90.83%
  Latency Distribution
     50%  101.38ms
     75%  107.35ms
     90%  117.77ms
     99%  304.69ms
  2700 requests in 15.10s, 752.48KB read
Requests/sec:    178.76
Transfer/sec:     49.82KB
```

#### 40 connections - 40 threads
- Test duration: 15.10s
- Total connections: 40
- Total number of requests: 5215
- Latency distribution: 125.54ms for the 90% percentile
- Latency distribution: 415.63ms for the 99% percentile
- Socket errors: 0

``` bash
Running 15s test @ https://cah-api-test.herokuapp.com/pick
  40 threads and 40 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   114.62ms   52.90ms 577.74ms   95.13%
    Req/Sec     9.34      2.30    20.00     89.61%
  Latency Distribution
     50%  102.18ms
     75%  108.39ms
     90%  125.54ms
     99%  415.63ms
  5215 requests in 15.10s, 1.42MB read
Requests/sec:    345.29
Transfer/sec:     96.16KB
```

#### 80 connections - 80 threads
- Test duration: 15.10s
- Total connections: 80
- Total number of requests: 7871
- Latency distribution: 465.44ms for the 90% percentile
- **Latency distribution: 1,35sms for the 99% percentile**
- Socket errors: 0

``` bash
Running 15s test @ https://cah-api-test.herokuapp.com/pick
  80 threads and 80 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   207.71ms  247.18ms   2.33s    89.45%
    Req/Sec     8.44      2.83    20.00     78.28%
  Latency Distribution
     50%  114.41ms
     75%  154.82ms
     90%  465.44ms
     99%    1.35s
  7871 requests in 15.10s, 2.14MB read
Requests/sec:    521.24
Transfer/sec:    145.04KB
```

### Conclusion for the regular customer reaslistic load, scaling number of threads and open connections
Observations:
- No socket connect errors were generated, within a max of 80 threads and 80 open connections.
- The latency distribution at 90% percentile is acceptable (125.54ms) until the 40 connections - 40 threads test scenario.
- The test scenario for 80 connections and 80 threads can be considered as failed:
- - Network latency at 90% percentile is almost four times higher, 465.44ms, then the previous test scenario.
- - Network latency at 99% can be considered the start of critical response time, above the 1 second range.
- The number of threads scales, together with the number of open connections during the whole testing execution. Simulating a scaled users spike.

## Loader.io test results
Loader.io was used to simulate both very high number of requests (10,000) over a period of 15 seconds, and more realistic scenarios like 80 users connection over the same period of 15 seconds.

### Test results for the scenario where 80 users connect over the period of 15 seconds

#### Test Results
Loader.io showing an average of 6ms response time:
![loaderTestResults](https://user-images.githubusercontent.com/1813225/156927112-32f81bc9-ea55-43df-b8ef-a4401c57feaa.png)


#### Bandwidth
Loader.io Bandwidth chart for the 80 user test scenario:
![loaderTestResultsBandwidth](https://user-images.githubusercontent.com/1813225/156927030-41057832-aedf-42a7-ac2a-64a7ba4c2094.png)

#### Request details
Loader.io Request details chart for the 80 user test scenario:
![loaderRequestDetails](https://user-images.githubusercontent.com/1813225/156927074-dc991ee9-4085-4361-92ee-8ab07afb773b.png)

### Test results for the scenario where 10,000 users connect over the period of 15 seconds
Loader.io showing an average of 27ms response time, without network errors:
![loader10kResults](https://user-images.githubusercontent.com/1813225/156928044-21bf9a05-bfce-4593-8982-21fc6050d6c2.png)

### Observations from the Loader.io performance tests
- The SUT can support, without errors and within an adequate response time, a spike of traffic of 10,000 connected user over a period of 15 seconds.
- The response time for a "slower" and more realistic scenario, e.g. 80 users connected over a period of 15 seconds, the SUT average response time was stable around 6ms, despite initial traffic spike.
- Monitoring via Heroku dashboard shows different response times during the high burst of traffic - 10,000 users.

#### Heroku Dashboard data during a high burst of traffic
Response time and troughput:
![herokuMetrics10000](https://user-images.githubusercontent.com/1813225/156928419-9a60d336-9c3c-42bf-bdc9-25877b781377.png)

-------
## Final conclusion
Considering the wrt test results and the metrics presented at the Heroku dashboard, it was proven that the system thresholds (without errors) currently are:
1. 10 threads and 200 connections, with a response time of 302ms for the 90% percentile
2. Considering the scenario where the number of clients (threads) increase with the same as the number of open connections, the threshold limit for request without a 3s timeout was for 500 threads and 500 connections. While using a load of 600 threads, the timeouts start to occur. 

Details from a 500 threads test:
``` bash
  500 threads and 500 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   253.45ms   78.43ms   1.16s    74.51%
    Req/Sec     4.04      1.99    10.00     79.47%
  Latency Distribution
     50%  240.16ms
     75%  286.22ms
     90%  365.24ms
     99%  516.94ms
  28032 requests in 15.11s, 7.63MB read
Requests/sec:   1855.64
Transfer/sec:    517.18KB
```

Details from a 600 threads test, with timeouts occurring:
``` bash
  600 threads and 600 connections
  Thread Stats   Avg      Stdev     Max   +/- Stdev
    Latency   482.71ms  408.60ms   3.00s    88.92%
    Req/Sec     2.86      2.29    10.00     84.41%
  Latency Distribution
     50%  337.76ms
     75%  547.06ms
     90%  936.48ms
     99%    2.40s
  17754 requests in 15.11s, 4.83MB read
  Socket errors: connect 0, read 0, write 0, timeout 149
Requests/sec:   1175.16
Transfer/sec:    327.47KB
```

