# EC500 Heart Monitor Mock application

## Prerequisites

1. Python 3+

## Run instructions
```python main_app.py```


## Exercise 1 - Asynchronous technique analysis

In this design we modeled the heart monitor as the composition of five separate threads 
of execution

1. BloodOxyGenSensorReader
2. BloodPulseSensorReader
3. BloodPressureSensorReader
4. RealtimeDataProcessor
5. PredictionEngine

Sensor reader threads periodically sample data from the sensor hardware and forward data to
both a database backend and the real time data processor. To more realistically model the
use-case, we treat displaying data to the terminal has a low latency task which should
happen immediately. In contrast, the real time data processor is not require to process
data in such a fashion; the sensor reader stores data in a queue which can be consumed
at an arbitrary pace. This design has pros and cons:

##### Queue based async design PROS

- Can model schedules in which tasks can be queued for later processing (and possible re-enqueued later)
- Priorities can be added on top of the default queue mechanism. For example, we can add a policy
wherein the blood pressure sensor samples are given the highest priority at a certain time
due to a patient's condition; this would allow the real-time data processor to prioritize
monitoring and notification on those data points
- Allows for load balancing of workloads: if the frequency of data entering the queue is too high
a system designer can improve the throughput of the application by adding more worker/slash consumer
threads


##### Queue based async design CONS

- Need to worry about synchronization: multiple threads accessing the queue can lead to funny
race conditions where overflow or underflow might happen
- Capacity worries: queue capacities are bounded by the size of the process address space, therefore
it is tricky to decide on a good capacity value if a fixed queue is used. If the queue is resizeable,
there is still the concern of overflow if the consumer threads are not quick enough or their state
prevents data consumption
- There is a design impact on the structure of the source code; developers needs to decide on the
best way to pull data from the queue (catch an exception vs query the state of the queue).