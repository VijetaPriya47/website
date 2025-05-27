+++
title = "How to Use Task Topology Plugin"
description = "Guide for using the Task Topology plugin in Volcano"
date = 2023-03-01
lastmod = 2023-03-01

draft = false  # Is this a draft? true/false
toc = true  # Show table of contents? true/false
type = "docs"  # Do not modify.

# Add menu entry to sidebar.
linktitle = "Use Task Topology Plugin"
[menu.docs]
  parent = "user-guide"
  weight = 17
+++

# Task Topology Plugin User Guide

## Environment setup

### Install volcano

Refer to [Install Guide](../../installer/README.md) to install volcano.

### Update scheduler configmap

After installed, update the scheduler configuration:

```shell
kubectl edit configmap -n volcano-system volcano-scheduler-configmap
```

Register `task-topology` plugin in configmap

```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: volcano-scheduler-configmap
  namespace: volcano-system
data:
  volcano-scheduler.conf: |
    actions: "enqueue, allocate, backfill"
    tiers:
    - plugins:
      - name: priority
      - name: gang
      - name: conformance
    - plugins:
      - name: drf
      - name: predicates
      - name: task-topology
        arguments:
          task-topology.weight: 10
      - name: proportion
      - name: nodeorder
      - name: binpack
```

### Running Jobs

Take tensorflow job as sample.

#### Install kubeflow/tf-operator

Refer to [Install Guide](https://www.kubeflow.org/docs/started/getting-started/) to install kubeflow, tf-operator included.

#### Edit yaml of tfjob

1. add annotations in volcano job or tensorflow job in format below.
   1. `affinity` annotation indicates that tasks have connections between each other, so they should be set on same nodes;
   2. `anti-affinity` annotation indicates that tasks do not have connections between each other, so they should be set on different nodes;
   3. `task-order` annotation indicates the order that tasks should be allocated. For example, `ps,worker` means scheduler should schedule `ps` tasks first. After all `ps` tasks were allocated, scheduler started to schedule `worker` tasks. **This annotation is not a required field.**

        ```yaml
            volcano.sh/task-topology-affinity: "ps,worker;ps,evaluator"
            volcano.sh/task-topology-anti-affinity: "ps;worker,chief;chief,evaluator"
            volcano.sh/task-topology-task-order: "ps,worker,chief,evaluator"
        ```
