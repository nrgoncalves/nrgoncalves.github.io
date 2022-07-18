---
title: "PySpark"
date: 2022-07-13T11:21:15+01:00
draft: false
categories: ["Articles"]
tags: ["PySpark", "Distributed"]
---

Concepts:

-   Spark application: a program that makes use of Spark.
-   Spark session: an object which exposes the Spark APIs to the developer. This is automatically created in interactive shells and it is assigned to the variable `spark` by default
-   Job: a computation that is triggered by a Spark action. Spark divides a Job into multiple Stages, representing it as a DAG which expresses dependencies between Stages.
-   Stage: a part of a Job that can be divided into smaller Tasks which can be carried out independently from one another
-   Task: A unit of computation to be executed by a Spark executor. It is sent to a particular core on a particular executor and will act on a single partition of data (i.e. a task does not operate on multiple partitions of data).

Spark does 2 types of operation on distributed data: transformations and actions. Transformations are lazily evaluated and they never change the original data in memory---instead, they generate new data, treating the input data as immutable.

Lazy evaluation allows spark to rearrange the computations in an efficient way prior to execution. Spark keeps track of the lineage of transformations and then rearranges them for efficiency.

Actions triggers lazy evaluation of the upstream transformations:

-   lazy evaluation: code optimisation
-   lineage and immutability: fault tolerance---replay lineage provides same results.

Transformation examples:

-   orderBy
-   groupBy
-   filter
-   select
-   join

Action examples:

-   show
-   take
-   count
-   collect
-   save
-   read

Narrow transformations: a single output partition can be computed from a single input partition
Wide transformations: a transformation which requires data or outputs across partitions (data shuffling). For instance, groupBy or orderBy are wide transformations because they require Spark to access data across partitions.
