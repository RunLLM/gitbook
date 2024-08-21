---
description: or, why MLOps set us down the wrong path
---

# The Aqueduct Philosophy

In recent years, MLOps has tied itself into knots — service ”architectures” that are more like a jumble of ad hoc connections between siloed, incomplete solutions to ML infrastructure. This leaves teams without a way to know what machine learning tasks are running, whether they’re working as expected, and how to triage and resolve issues. Ultimately, it means that less time is spent on delivering on the promise of machine learning — the bulk of time is spent wrangling and caring for a knot of intertwined infrastructur

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

Infrastructure complexity is not inevitable, but complexity does grow exponentially as tools are added. Systems like Apache Spark, Tensorflow, PyTorch, and Ray have enabled countless new machine learning use cases, and each one of these tools is quite powerful. Unfortunately, the proliferation of these exact tools incurs a significant maintenance overhead and learning curve that’s led to the MLOps knot. It’s effectively slowed progress in the value delivered by machine learning.

Over the last year, our team has met with 250+ enterprises to discuss the challenges around ML infrastructure. The typical enterprise I’ve interviewed might have a stack that looks something like this:

* Snowflake as a data & feature warehouse with Fivetran + dbt for ELT
* Spark (or Databricks) for large scale analysis and lightweight feature computation
* Ray for experimentation and hyperparameter tuning
* Weights & Biases for experiment management
* Airflow for orchestrating batch training & inference

This is a long list of tools to maintain, and this list already has some painful (and often expensive) redundancy — e.g., compute on Snowflake vs. Databricks, metadata split across dbt and W\&B. But it rarely stops there since this just covers the basics — [things get much more complicated](https://www.mihaileric.com/posts/mlops-is-a-mess/) when you include distributed Python featurization (Dask, Modin), feature stores (Feast, Tecton), model performance monitoring (Truera, Arize), or real-time prediction serving. This has led to two key challenges that I’ve heard time and time again.

_**First**_**,** **infrastructure proliferation**. Maintaining existing systems and adopting new ones is extremely painful. Each of these systems presents a new API & interface for data science & ML engineers to learn. With 7-10 systems for managing production ML, no one person can possibly be fluent in all the relevant APIs, much less manage each one of these systems in production. The work required to run ML in production eats up a significant chunk of time, often requiring a new team to be formed. That can obviously be frustrating, but at least it’s a one time-learning curve.

Worse yet, when a new ML task comes along that has new requirements — e.g., Ray or Dask for distributed Python feature computation — every workflow has to be rewritten from the existing API to support the new library. So much time and effort has been sunk into the existing infrastructure that learning about and evaluating a new system is a painful, weeks-long project (or a non-starter).

And as always, switching between cloud providers is 100x harder still.

_**Second**_**, metadata drift**. many teams have trouble knowing what code is running where, whether it’s succeeding or failing, and who’s responsible for it. The challenge comes from the lack of shared context and interoperability within these systems, many of which might have overlapping functionality.

Ultimately, there’s no source of truth for the code, data, and metadata that’s moving across these systems. Code & data artifacts remain relatively constant across the ML lifecycle, but there is no shared context across systems. For example, the same code will be used for experimentation in Ray and eventually for production training in Kubeflow. The resulting model might then be run on Databricks for large-scale batch inference or deployed into a separate system for real-time inference — again. At each stage, the code, the model, and the resulting data are repackaged into different APIs and different formats, which is both time-consuming and — critically — loses context by moving across systems.

### Why Aqueduct?

Aqueduct is focused on untangling the MLOps Knot by building a single interface to run machine learning on your existing cloud infrastructure.

Aqueduct isn’t building a “better” component for any of the stages of the lifecycle. Instead, we’re building unifying, ML-centric APIs that integrate with and empower industry-standard components.

Aqueduct [integrates natively with your existing tools](resources/), allowing you to run ML seamlessly in your cloud — we have support for Kubernetes, Airflow, AWS Lambda, and Databricks, with Ray and others to follow. Once your code is running with Aqueduct, you automatically get visibility into whether things work and what’s happening (logs, stack traces, metrics). All of this metadata is organized in a single interface, regardless of whether you’re running on one piece of infrastructure or ten.

Aqueduct has a simple, Python-native interface that allows you to define ML tasks, get them running quickly, and move across infrastructure as needed. Whether that’s going from your laptop to the cloud or from AWS to GCP, Aqueduct has single, Python-native API that gives you the flexibility to choose the best tools for the job.

### What's Next?

* Check out our [open-source project](https://github.com/aqueducthq/aqueduct).
* Join the [community conversation](https://slack.aqueducthq.com)!

_This page has been adopted from our blog posts on_ [_the MLOps Knot_](https://aqueducthq.com/post/the-mlops-knot/) _and_ [_Untangling the MLOps Knot_](https://aqueducthq.com/post/untangling-the-mlops-knot/)_._
