# Practical Session #1: Introduction

1. Find in news sources a general public article reporting the discovery of a software bug. Describe the bug. If possible, say whether the bug is local or global and describe the failure that manifested its presence. Explain the repercussions of the bug for clients/consumers and the company or entity behind the faulty program. Speculate whether, in your opinion, testing the right scenario would have helped to discover the fault.

2. Apache Commons projects are known for the quality of their code and development practices. They use dedicated issue tracking systems to discuss and follow the evolution of bugs and new features. The following link https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-794?filter=doneissues points to the issues considered as solved for the Apache Commons Collections project. Among those issues find one that corresponds to a bug that has been solved. Classify the bug as local or global. Explain the bug and the solution. Did the contributors of the project add new tests to ensure that the bug is detected if it reappears in the future?

3. Netflix is famous, among other things we love, for the popularization of *Chaos Engineering*, a fault-tolerance verification technique. The company has implemented protocols to test their entire system in production by simulating faults such as a server shutdown. During these experiments they evaluate the system’s capabilities of delivering content under different conditions. The technique was described in [a paper](https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf) published in 2016. Read the paper and briefly explain what are the concrete experiments they perform, what are the requirements for these experiments, what are the variables they observe and what are the main results they obtained. Is Netflix the only company performing these experiments? Speculate how these experiments could be carried in other organizations in terms of the kind of experiment that could be performed and the system variables to observe during the experiments.

4. [WebAssembly](https://webassembly.org/) has become the fourth official language supported by web browsers. The language was born from a joint effort of the major players in the Web. Its creators presented their design decisions and the formal specification in [a scientific paper](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf) published in 2018. The goal of the language is to be a low level, safe and portable compilation target for the Web and other embedding environments. The authors say that it is the first industrial strength language designed with formal semantics from the start. This evidences the feasibility of constructive approaches in this area. Read the paper and explain what are the main advantages of having a formal specification for WebAssembly. In your opinion, does this mean that WebAssembly implementations should not be tested? 

5.  Shortly after the appearance of WebAssembly another paper proposed a mechanized specification of the language using Isabelle. The paper can be consulted here: https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf. This mechanized specification complements the first formalization attempt from the paper. According to the author of this second paper, what are the main advantages of the mechanized specification? Did it help improving the original formal specification of the language? What other artifacts were derived from this mechanized specification? How did the author verify the specification? Does this new specification removes the need for testing?

## Answers

### RegreSSHion (CVE-2024-6387)

The regreSSHion vulnerability is an unauthenticated remote code execution flaw found in OpenSSH servers (sshd) on glibc-based Linux systems. If exploited, it allows full root access to the targeted machine without user interaction. This vulnerability is classified as High severity. 

RegreSSHion is an 2006 issue that have been reintroduce in 2020. 

This regression was introduced in October 2020 (OpenSSH 8.5p1) by commit 752250c (“revised log infrastructure for OpenSSH”), which accidentally removed an ”#ifdef DO_LOG_SAFE_IN_SIGHAND” from sigdie(), a function that is directly called by sshd’s SIGALRM handler. 

This bug has been discovered in 2021 by Qualys Threat Research Unit.

This issue is triggered when a client fails to authenticate within the LoginGraceTime period (default 120 seconds). When this timeout occurs, sshd’s SIGALRM handler is called asynchronously, invoking functions that are not safe to use in signal handlers, such as syslog().

This bug is global since multiple ssh server could have been target while the bug was present (between Oct 2020 and 2024). This bug could have been exploited to run code as root remotely.

This issue shouldn’t have been reintroduce. When the bug was discovered in the 2006, they didn’t used regression testing to prevent this regression. 


Source: [Wikipedia](en.wikipedia.org/wiki/RegreSSHion), [Logpoint](https://www.logpoint.com/fr/blog/lhistoire-de-regresshion-une-vulnerabilite-sshd-qui-refait-surface/), [NVD](https://nvd.nist.gov/vuln/detail/CVE-2024-6387), [Qualys Threat Research Unit](https://www.qualys.com/2024/07/01/cve-2024-6387/regresshion.txt)

### Apache Common Bug

*The Bug*

This issue is about counter of a MultiSet that doesn’t update when removing the last element.

This bug is global since it affects all programs that use `MultiSet` in apache `commons-collections`

```java
protected int getCountAfterRemoval(MultiSet<String> multiset) {
  MultiSet.Entry<String> entry = multiset.entrySet().iterator().next();
  entry.getCount(); // = 2
  multiset.remove(entry.getElement());
  entry.getCount(); // = 1
  multiset.remove(entry.getElement());
  return entry.getCount(); // Still = 1, should be 0
}
```

*The Solution*

This bug has been resolve by a pull request from `CasualSuperman`.

He modified `public int remove(final Object object, final int occurrences`, by adding this line `mut.value = 0` when the multiset has only one object remaining. 

He also added a new test to prevent future regression. 

This bug is global since it involve public methods of the library.

Source: [Bug](https://issues.apache.org/jira/browse/COLLECTIONS-709?jql=project%20%3D%20COLLECTIONS%20AND%20statusCategory%20%3D%20Done%20AND%20type%20%3D%20Bug%20ORDER%20BY%20updated%20DESC), [Pull request](https://github.com/apache/commons-collections/pull/66/filess)

### Netflix Chaos Engineering

Netflix has million of users across the world, and a single machine is not enough to host all the services provided by the platform. That’s why Netflix system is distributed across multiple servers that have to coordinate over the network. Thus, a traditional software testing approach is not sufficient to identify failures. 

The possible failures based on Netflix observations can be :

    The overload of a server, causing the clients requests to be places on a queue. Over time, the queue consumes more and more memory, causing the client to fail.
    A client makes a request to a service that is fronted by a cache. The service returns a transient error which is incorrectly cached. When other clients make the same request, they are served an error response from the cache.


These failures are only reproducible in production. 

Netflix, with the chaos engineering method, performed concrete experiments such as :

    terminate virtual machine instances (Chaos monkey)
    inject latency into requests between services (latency monkey)
    fail requests between services
    fail an internal service
    make an entire Amazon region unavailable (by redirecting users to others amazon region, Netflix has not the capability to take an entire amazon region, simulate chaos kong)


In other case, a Netflix service can behave properly for users, but act like unavailable for a subset of users.

To do such tests, the system must be able to continuously deliver the service to clients, despite the errors and failures introduced.

The concept of chaos engineering is new and at the date of the paper, only Netflix seems to use it. 
But, this method of testing could be used in all other big distributed systems (“Internet scaled organizations”), like amazon, google… 

### WASM Part 1 

The formal specification does not mean that the specification or the implementation is formally proven (Constructive approach), thus we cannot guarantee the absence of bugs, thus we need to do testing to try and find the presence of bug and fix them. 

The formal specification is mainly useful for implementing the VM and the compiler (eg. from C/C++/OCaml) on different target (arch-os).

They did a reference implementation of the specification in Ocaml which allow a close to the formal specification implementation. They used this reference interpreter to run extensive test suite. And test production implementations (on different browser) and reference specification as well as to prototype new features.  

We can see that over the "formal specification" they did extensive testing.

### WASM Part 2

This time they used a Constructive approach to prove the specification, they used Isabelle to do the proof. Before no test where done directly on the specification, they were done on the implementation side. Proving the specification is only a layer of validation over the first approach(#WASM Part 1).   

The mechanized specification allowed to find flaws directly inside the Web Assembly specification (by Constructive approach) and not on the implementation of the specification itself.

In the course of conducting these proofs, They have identified and assisted in fixing several errors in the official WebAssembly specification.

They created an other interpreter and type checker from the the mechanized specification (with Isabelle). But they prove the parser and linker, they used the reference parser and linker to use their interpreter and type checker. So it still not pure. Thus since not every part of the program is not made by Constructive approach we cannot prove the absence of bugs. Also we still need to test the production implementation. So the  mechanized specification does not remove the need of testing. 



