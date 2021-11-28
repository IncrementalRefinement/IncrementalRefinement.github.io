---
layout: post
title: MapReduce
tags: [Distributed System, MapReduce]
---

This notes is written after I read the classic paper---"MapReduce: Simplified Data Processing on Large Clusters". It aims to summarize my understanding as well as pave the way for my work of finishing MIT6.824 lab1

## 1. Overview

I think the term MapReduce can refer to multiple concepts:

1. The idea of processing a large amount data in a divide-and-conquer-like way in parallel
2. The distributed system that is capable of performing the idea in a hardware level(cluster of commodity machines)
3. The software framework the hide the complexity of managing the distributed system and let the programmer program in a rather easy way.

The typical situation is as follows:  Users specify a map function that processes a key/value pair to generate a set of intermediate key/value pairs, and a reduce function that merges all intermediate values associated with the same intermediate key. And all these are usually accomplished with the well-refined MapReduce framework and executed on a cluster of the commodity machines.

## 2. Implementation

I think it would be helpful to have a basic awareness of the concept of Task and participants before understanding the overall architecture. 

### 2.1 Tasks

As we indicated in the previous section, the idea of MapReduce is similar to the idea divide-and-conquer in algorithm design. The task here can be generalize into two kinds:

1. Map, which is done before Reduce. Before mapping, the input data were firstly partitioned into unrelated parts and thus can be processing in parallel and generate the intermediate K-V pairs for the reduce task.
2. The worker of the Reduce task will take in the intermediate K-Vs generated before and output the final result of this MapReduce procedure. The output result could be used as the input for another round of MapReduce

*  The input keys and values are drawn from a different domain than the output keys and values. Furthermore, the intermediate keys and values are from the same domain as the output keys and values.

### 2.2 Participants

1. Master: the master dominates the functioning of the distributed system, including work scheduling, failure detection......Take it as the administrator and supervisor.
2. Worker: perform the work of map&reduce

### 2.3 The Overall Architecture

Google presented the straightforward and elegant architecture they accomplished in the paper. In this architecture, Master is responsible for the well-functioning of the whole system and has an overall point of view of the whole system. The worker takes their computational responsibility under the command of master and only have limited information from their own perspective. The master thus becomes the single point of the system and the mechanism within it should be well designed. Also, the google introduced many design ideas to make the system having better performance, being more fault-tolerant by keeping some degree of redundancy intentionally and utilizing the locality of information storage and processing.

## 3. Refinements

The refinement of the implementation can be found on the paper and many other materials, so I'm not gonna do overtalk here. The refinements can be achieve in many ways:

1. Partitioning function
2. Making ordering guarantees
3. Combiner function that do partial merging before network flow
4. ......

## 4. Practice and Performance

The paper is written in 2004. At tha time, google has adopted the idea of MapReduce and widely made use of it in many circumstances such as machine learning, large scale computation and so on, in replace of the traditional ar-hoc way. The framework works pretty well and make many project more efficient and easy to maintained than the previous approach of the ad-hoc distributed passes in the prior version. Looking backward at the end of 2021, the MapReduce has promoted the development of big data processing, and inspired many distributed computation framework.

## 5. My Random Thoughts

One interesting point of the MapReduce framework is that it hides the complexity of managing a distributed system away from the programmer who used it to processing data. By doing that, the freedom is somewhat deprived because of the restriction on the program model, but the work is also simplify and the project can be more efficiently delivered. The framework is some kind of low-code attemption, and i think the degree of low-coding should be considered carefully to achieve the best equilibrium between freedom and simplicity.
