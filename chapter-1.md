# Reliable, Scalable, and Maintainable Applications

Reliability: The system should continue to work correctly even in the face of adversity
Scalability: As the system grows, there should be reasonable ways of dealing with that growth
Maintainability: Over time, many different people will work on the system, and they should all be able to work on it productively

## Reliability

Faults can be in hardware, software, and humans. Fault-tolerance can hide certain types of faults from the end-user.

## Scalability

### Describing Performance

Latency and response time are not the same thing. Response time is what the client sees, including the actual time to process the request (service time) as well as network delays and queueing delays. Latency is the duration that a request is waiting to be handled.

It is usually better to measure in percentiles. For instance, the median response time would indicate that half of all users experience a response time shorter than the median and half of all users experience a response time longer than the median. p95, p99, p999 can also be percentile metrics worth capturing.

It may be a good idea to keep a rolling window of response times in the last ten minutes. Every minute, you calculate the median and various percentiles over the values in that window and plot those metrics on a graph.

### Coping with Load

Scaling up: vertical scaling, moving to a more powerful machine
Scaling out: horizontal scaling, distributing the load across multiple smaller machines. This is also known as a shared-nothing architecture

## Maintainability

Good abstractions can help reduce complexity, and increase the evolvability of a system. Good operability requires insight into the system's health, and having effective ways of managing it
