---
layout: page
title:  "Writing a backend"
permalink: writing-a-backend
---

#### Starting point: provide an ExecutionContext

Your back-end entry point is the implementation of the `ExecutionContext` interface
* Many complicated functions
* Fortunately, you can just extend from `AbstractExecutionContext`
  * Just a single `instance()` method left virtual, basically a call to the constructor.
  * You can see how’s implemented in the simulator for a clue


#### Expose device capabilities

The ExecutionContext also holds device capabilities (it’s like the `System` / `Runtime` singletons of Java).
* A number of interfaces for standard capabilities are provided
  * TimeAwareDevice — Implement this if your device is able to estimate network message delays and lags
  * SpatiallyEmbeddedDevice — Implement this if your device is able to estimate distances to its neighbors (with whatever metric)
  * LocalizedDevice — If your device has access to its position (e.g., if a GPS system is onboard)
* You are free to add any method, and access it via calls to self
* The thing usually implemented is something along the line of `MyExecutionContext extends AbstractExecutionContext implements TimeAwareDevice, LocalizedDevice`

#### Implement communication

Communication is dealt with by the `NetworkManager`
* `void shareState(Map<CodePath,Object> toSend)`
  * You are in charge of finding a way to send your neighbors the `toSend` payload...
* `Map<DeviceUID,Map<CodePath,Object>> getNeighborState()`
  * ...and of presenting the payloads sent from others to the current device.

#### Run

Now that pieces are in place, just create a ProtelisVM and run whatever
you like, e.g.:
```java
final NetworkManager netmgr = new MyNetworkManager(...);
final ExecutionContext ctx = new MyExecutionContext(netmgr, ...);
final ProtelisProgram prog = ProtelisLoader.parse(myProgram)
final ProtelisVM vm = new ProtelisVM(prog, ctx);
while (alive) {
  vm.runCycle();
  Thread.sleep(someTime);
}
```
