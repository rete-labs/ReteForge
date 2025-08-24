# ReteForge

Where distributed systems are forged and tested

## About

_Rete means Network in Latin._

ReteForge implements automated deploy and execution environment of distributed systems for development, testing, benchmarking and demo purposes. With main focus on new network technology as we're developing one in ReteLabs.

ReteForge isn't network simulator nor emulator but rather multi-purpose environment to support developing of distributed systems.

## Target audience

The project is mainly intended for developers of distributed systems who focus on prototyping and eventual production-ready implementations.

By distributed systems we mean both applications and core technologies like networks and databases. Examples of applications:
- Alice made beautiful web app and wants to publish it to the network. She uses ReteForge to publish it to a local testnet and run a bunch of standard test suites that check website behaviour under varying network conditions.
- Bob works at startup that builds a web app consisting of a bunch of microservices. Bob needs to check his app resilience to the downfall of some of them.

Examples of system-level software:
- Carol is working on a new post-IP network. She uses ReteForge to deploy and test her implementation under different conditions, and also to compare it with IP-based legacy solution.
- Dave made new leaderless distributed database. He needs to benchmark it under network splits and joins.
- Eve works on a new service mesh project. She uses ReteForge to test how the mesh balances loads when some links fail.

ReteForge also may be of interest to researchers in networking and distributed systems, but its usefulness may be limited there as we're not developing a scientific instrument but engineering one, so we focus on practical aspects instead of purity of experiments and theoretical analysis.

There are a lot of network simulators (NS-3, OmniNet++ to name some) that seem more suitable for researchers. On the other hand, we don't know any public software with goals similar to ReteForge, that's why we started this project.

## Project status

ReteForge is in planning and early development phase.

## Initial use cases

With these use cases we start our development. They're network-focused as we're using ReteForge to facilitate development of a new networking technology.

### Demo of the adaptable mobile network

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
   - Parameters (e.g. link quality) can be interactively adjusted. This enables quick, ad hoc experiments
4. Metrics collected
   - Emulated host movements
   - Emulated link quality
   - How network under test perceives the situation: its link quality and other metrics, especially related to handover event
   - Connection quality between Alice and Bob - as reported by the network and by themselves
   - Packet dumps for detailed analysis
   - CPU, memory usage, IOPS etc
5. Visualization
   - Hosts and layers of links between them (3D diagram is the best, with possibility to select which layers to show; we can start with 2D though)
   - Graphs of the metrics (with ability to select which to show)
   - Visualization of the connection between Alice and Bob (can be video stream that is demonstrated)
6. Comparison of results
   - It should be possible to compare results of different executions - both different versions of the same network (to see possible improvements or degradation) and different network technologies. For us, it is crucial to compare to legacy IP network: how both handle the same input data, same failure scenarios, same load patterns, and so on. For example, ReteForge collects real-time comparison of metrics: rtt, goodput, resource usage, then reports: "under certain packet loss conditions, our network maintained 95% performance while IP dropped to 60%"

### Benchmarking

Basic scenario for benchmarking - implement one link between hosts A and B, and collect metrics from it under various loads. Compared to Demo case, Benchmarking adds a need for multiple runs in the same environment.

There are 2 ways to run a bench:
- external: network software under test receives load from external traffic generator (e.g. via socket or tun device)
- native: traffic generator is built into special benchmarking executable with network under test as a library

The goals of our benchmarking:
1. Explore: better understand how implemented functionality works. Even links are very complex software, may be integrated as a few layered protocols implemented by different vendors. We need to test them under various conditions, identifying performance patterns and discovering unexpected interactions
2. Regression control: an automated way to ensure that during development the performance of the resulting implementation doesn't degrade
3. Comparison to reference benchmarks: a way to be sure the implementation isn't too bad in terms of performance (during prototyping we're not striving for performance, but we mustn't pessimize either; currently we allow 10% worse performance than references, but more than that is undesirable)
4. Performance tuning: an environment where we can play with link configurations to see how they affect resulting metrics - and how we can make link work better (and later - how to tune it by automatic control system of the network)
5. Link behaviour under changing quality of the underlying media: a way to look how link maintains itself with loss, delay etc.

Let's break down what we want from ReteForge for benchmarking:
1. Automated deployment
   - Hosts are created by ReteForge (emulated as containers or VMs) or adopted (physical hosts)
   - Emulated links with configurable bandwidth, loss, delay (static config seems good enough for now)
   - Network software under test is deployed to these hosts and properly configured
   - Benchmark software is deployed to these hosts and properly configured (this includes native benchmarks, 3rd-party traffic generators, 3rd-party reference benchmarks - also native and external)
2. Execution
   - Multiple runs in different modes: native/external/reference, emulated hosts/physical hosts/mix of them etc
   - Long runs to identify possible degradation (leaks, overflows etc)
3. Monitoring
   - Execution environment must be maintained pure during benchmarking to avoid influence from unwanted factors like software in background (e.g. auto-updater runs during the test; for physical hosts 3rd party network activity may affect the results as well)
4. Result collection and analysis
   - Monitored metrics saved to files (CPU loads, mem usage etc)
   - Reported metrics saved to files
   - Packet dumps, if requested
   - Graphs of selected metrics shown
   - Automatic check of the results for regressions
   - Automated anomaly detection

Currently we perform "benchmarking without feedback", this means Alice and Bob do not communicate aside of simple reflection of packets in some benches. Advanced benchmarking may require collaboration between Alice and Bob - and instead of doing this in-band (i.e. within benched link), we should implement it out-of-band, via means provided by ReteForge's environment. This is advanced topic to explore later.

## Future use cases

### Deploying of public testnet

We need to host public nodes of our network to facilitate forming of the open source community. Ideally, an enthusiast should be able to build our network platform (or even install prebuilt image, e.g. a container or mobile app) and connect to the public network to see how it works. In more advanced scenario, he should be able to form a local network and connect it to the public network.

What can be demonstrated:
- Hosting of Web services (like websites) in the network
- Public network as a federation of user's networks
- VPN-like scenarios: automatic connectivity of user's network parts over public network
- Mobility scenarios: e.g. device moves from one Wifi network to another, changing its IP address, but connectivity to some service isn't interrupted
- Multihoming scenarios: e.g. device is connected to network by few links, and all of them are utilized by single connection

Demo scenarios should be worked upon, but overall the need for public testnet seems to be very well justified already.

So, in this scenario we want following:
1. Automated deployment
   - Hosts are created by ReteForge (emulated as containers or VMs) or adopted (VPS, physical hosts)
   - Cross-datacenter, maybe even cross-provider deployment (so we have network agents in different envirnoments)
   - Network software is deployed to these hosts and properly configured
   - Scaling: ability to add or remove hosts from operating environment, including automated one (e.g. in a pre-configured range)
2. Monitoring
   - The health of the public network infrastructure is monitored, alerting admins in case of outages or other incidents
3. Security
   - While testnet shouldn't be used for production, it is still our infrastucture that must be kept secure. Need to ensure it isn't abused nor used to abuse other systems in the Internet
   - Access to public testnet should be controlled (e.g. by user accounts), routed traffic - counted, abuse prohibited
   - Publishing of questionable content (broad topics like porn, violence etc) in public testnet should be prohibited (and offenders banned), with a proper way to report such content to our moderators

Obviously, implementing public testnet requires special features not only from ReteForge, but from network software itself, and around it. ReteForge should serve as the foundation for deploying and managing it.

### Use cases for distributed applications

We start development of ReteForge with our network use case in mind. But ReteForge should evolve from a specialized tool for network developers into a comprehensive platform for distributed systems deploying and testing.

These use cases to be determined.

## Implementation notes

ReteForge, being used to facilitate our development of new network technology, should be "dogfooding" it, i.e. eventually should be powered by it. It should become one of the first adopters of this new technology, and may play feature driver role for it.
