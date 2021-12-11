# implementation (bull & js)
![[bull.png]]
Bull is a Node library that implements a fast and robust queue system based on [redis](https://redis.io/).
### Lifecycle
![[bull-job-lifecycle.png]]
a job enters a lifecycle where it will be in different states, until its completion or failure (although technically a failed job could be retried and get a new lifecycle).
When a job is added to a queue it can be in one of two states:
###### state 1
`“wait”` status, which is, in fact, a waiting list, where all jobs must enter before they can be processed
`“delayed”` status: a delayed status implies that the job is waiting for some timeout or to be promoted for being processed, however, a delayed job will not be processed directly, instead it will be placed at the beginning of the waiting list and processed as soon as a worker is idle.

###### state 2
The next state for a job:
**active** state. The active state is represented by a set, and are jobs that are currently being processed, i.e. they are running in the **process function.** A job can be in the active state for an unlimited amount of time until the process is completed or an exception is thrown so that the job will end in either the `“completed”` or the `“failed”` status.

### Let's start
at the first you have to install, up and run `Redis`. 
```bash
npm install bull --save
# or
yarn add bull
```
**Bull** will by default try to connect to a **Redis** server running on `localhost:6379`
A queue is simply created by instantiating a Bull instance:
```js
const myFirstQueue = new Bull('my-first-queue');
```
A queue instance can normally have 3 main different roles: 
- A job producer
- a job consumer
- an events listener.

order: FIFO (the default), LIFO

### Producer
A job producer is simply some Node program that adds jobs to a queue
```js
const job = await myFirstQueue.add({
  foo: 'bar'
});
```
As you can see a job is just a javascript object. This object needs to be serializable

### Consumer
A consumer or worker is nothing more than a Node program that defines a process function like so:
```js
myFirstQueue.process(async (job) => {
  let progress = 0;
  for(i = 0; i < 100; i++){
    await doSomething(job.data);
    progress += 10;
    job.progress(progress);
  }
});
```
The `process` function will be called every time the worker is idling and there are jobs to process in the queue. Since the consumer does not need to be online when the jobs are added it could happen that the queue has already many jobs waiting in it, so then the process will be kept busy processing jobs one by one until all of them are done.

### Listener
you can just listen to events that happen in the queue. Listeners can be local, meaning that they only will receive notifications produced in the _given queue instance_, or global, meaning that they listen to _all_ the events for a given queue. So you can attach a listener to any instance, even instances that are acting as consumers or producers. But note that a local event will never fire if the queue is not a consumer or producer, you will need to use global events in that case.
```js
// Define a local completed event
myFirstQueue.on('completed', (job, result) => {
  console.log(`Job completed with result ${result}`);
})
```