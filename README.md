# ReteForge

Where networks are forged and tested

## About

The main goal of the project: implement automated deploy and execution environment of distributed applications for development, testing, benchmarking and demo purposes. With main focus on new "network stacks" as we're developing one in ReteLabs.

Note: we're not developing a scientific instrument but engineering one, so we focus on practical aspects instead of purity of experiments and theoretical analysis.

Rete means Network in Latin.

## Use cases

### 1. Demo of the adaptable mobile network

Let's start with the simplest networking demo possible. We have 3 mobile hosts A, B, C that are organized in a fully-connected network (links AB, BC, CA). There is Alice at host A that communicates with Bob at host B via the network. At the beginning connection uses direct link AB. Then mobile hosts A and B start moving away from each other and the link between them degrades in quality while their links to C are stable as it moves in between them. At some point, the connection is automatically switched to relay mode via C, i.e. AC-CB.

```
Alice   Bob
 |-------|
(A)-----(B)         Alice            Bob
 \      /             |---------------|
  \    /       =>    (A)-----(C)-----(B)
   \  /
    (C)
```

It is desirable that such handover happens as smooth as possible - and it is up to the network under test to show its capability, and for ReteForge - to implement the scenario and visualize it.

So, in this scenario we want following:
1. Automated deployment
   - Hosts are created by ReteForge (emulated as containers or VMs) or adopted (physical hosts)
   - Emulated links between the hosts are configured - with limited bandwidth, delay etc (e.g. IP links if we test new network in overlay mode)
   - Network software under test is deployed to these hosts and properly configured
   - Demo load software (Alice and Bob) is deployed to selected hosts and properly configured
2. Emulated link quality change
   - Link quality (delay, loss) change over time is configured. One way to specify the link quality over time is via emulated mobility: each host is configured with mobility mode (e.g. linear movement from X1,Y1,Z1 to X2,Y2,Z2 over duration of T), and link quality as function of distance between the hosts is specified.
3. Execution of the scenario
   - Demo load is started and emulation of physical events is performed in quasi-realtime
4. Metrics collected
   - Emulated host movements
   - Emulated link quality
   - How network under test perceives the situation: its link quality and other metrics, especially related to handover event
   - Connection quality between Alice and Bob - as reported by the network and by themselves
   - Packet dumps for detailed analysis
5. Visualization
   - Hosts and layers of links between them (3D diagram is the best, with possibility to select which layers to show)
   - Graphs of the metrics (with ability to select which to show)
   - Visualization of the connection between Alice and Bob (can be video stream that is demonstrated)

### 2. Link benchmarking

Basic scenario for benchmarking - implement one link between hosts A and B, and collect metrics from it under various loads. Compared to Demo case, Benchmarking adds a need for multiple runs in the same environment.

There are 2 ways to run a bench:
- external: network software under test receives load from external traffic generator (e.g. via socket or tun device)
- native: traffic generator is built into special benchmarking executable with network under test as a library

The goals of our benchmarking:
1. Regression control: an automated way to ensure that during development the performance of the resulting implementation doesn't degrade
2. Comparison to reference benchmarks: a way to be sure the implementation isn't too bad in terms of performance (during prototyping we're not striving for performance, but we mustn't pessimize either; currently we allow 10% worse performance than references, but more than that is undesirable)
3. Performance tuning: an environment where we can play with link configurations to see how they affect resulting metrics - and how we can make link work better (and later - how to tune it by automatic control system of the network)
4. Link behaviour under changing quality of the underlying media: a way to look how link maintains itself with loss, delay etc.

Let's break down what we want from ReteForge for benchmarking:
1. Automated deployment
   - Hosts are created by ReteForge (emulated as containers or VMs) or adopted (physical hosts)
   - Emulated links with configurable bandwidth, loss, delay (static config seems good enough for now)
   - Network software under test is deployed to these hosts and properly configured
   - Benchmark software is deployed to these hosts and properly configured (this includes native benchmarks, 3rd-party traffic generators, 3rd-party reference benchmarks - also native and external)
2. Execution
   - Multiple runs in different modes: native/external/reference, emulated hosts/physical hosts/mix of them etc
3. Monitoring
   - Execution environment must be maintained pure during benchmarking to avoid influence from unwanted factors like software in background (e.g. auto-updater runs during the test; for physical hosts 3rd party network activity may affect the results as well)
4. Result collection and analysis
   - Monitored metrics saved to files (CPU loads, mem usage etc)
   - Reported metrics saved to files
   - Packet dumps, if requested
   - Graphs of selected metrics shown
   - Automatic check of the results for regressions
   - Automated anomaly detection

Currently we perform "benchmarking without feedback", this means Alice and Bob do not communicate aside of simple reflection of packets in some benches. Advanced benchmarking may require collaboration between Alice and Bob - and instead of duing this in-band (i.e. within benched link), we should implement it out-of-band, via means provided by ReteForge's environment. This is advanced topic to explore later.
