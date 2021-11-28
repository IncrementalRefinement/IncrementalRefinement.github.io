---
layout: post
title: MapReduce
tags: [Distributed System, MapReduce]
---

This notes is written after I read the classic paper---"MapReduce: Simplified Data Processing on Large Clusters". It aims to summarize my understanding as well as pave the way for my work of MIT6.824 lab1

## 1. Overview

I think the term MapReduce can refer to multiple concepts

1. The idea of processing a large amount data in a dive-and-conquer-like way in parallel
2. The distributed system that is capable of performing the idea in a hardware level(mostly made up cluster of commodity machines)
3. The software framework the hide the complexity of managing the distributed system and let the programmer program in a rather easy way.

The typical situation is as follows:  Users specify a map function that processes a key/value pair to generate a set of intermediate key/value pairs, and a reduce function that merges all intermediate values associated with the same intermediate key. And all this is usually accomplished with the well-refined MapReduce framework and executed on a cluster of the commodity machines.

## 2. Implementation

The architecture has some concepts:

### 2.1 Tasks

As we indicated in the overview section, the idea of MapReduce is similar to the idea dive-and-conquer. The task here can be generalize into two kinds:

1. Map, which is done before Reduce. Before mapping, the input data were firstly partitioned into unrelated parts and thus can be processing in parallel and generate the intermediate K-V pairs for the reduce task.
2. The worker of the Reduce task will take in the intermediate K-Vs
// TODO: here

### 2.2 Participants

### 2.3 The Overall Architecture


## 3. Refinements

The refinement of the implementation can be found on the paper and many other materials, so I'm not gonna do overtalk here. The refinements can be achieve in many ways.

1. Partitioning function
2. Making ordering guarantees
3. Combiner function that do partial merging before network flow
4. ......

## 4. Practice and Performance

The paper is written in 2004, at tha time, google has adopted the idea of MapReduce and widely made use of it in many circumstances such as machine learning, large scale computation and so on, in replace of the tradition. The framework works pretty well and make many project more efficient and easy to maintained than the previous approach of the ad-hoc distributed passes in the prior version. Looking backward at the end of 2021, the MapReduce has promoted the development of big data process, and inspired many framework.

## 4. My Random Thoughts

