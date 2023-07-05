---
description: How all code runs in Aqueduct
---

# Operators

Every piece of code in Aqueduct is encapsulated within an **Operator**. Operators take in and return [**Artifacts**](artifacts.md). Currently, Aqueduct supports Python operators and data retrieval operators. We plan to add support for other langauges like R in the future.

This guide will walk you through:

* [Creating a Python Operator](operators/creating-a-python-operator.md)
* [Specifying a `requirements.txt`](operators/specifying-a-requirements.txt.md)
* [Adding File Dependencies in Python](operators/file-dependencies-in-python.md)
* [Improve Dependencies and Python Version Management Using Conda](resources/compute-systems/conda.md)
* [Eager vs Lazy Execution](operators/lazy-vs-eager-execution.md)
* [Configuring GPUs, CPUs, and Memory](operators/configuring-resource-constraints.md)
