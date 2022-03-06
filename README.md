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
Load spike: By measuring the performance metrics while creating a high quantity of connections (e.g. 10,000), the application response time and latency distribution are registered. Thus providing a load/stress baseline for traffic spikes. 
