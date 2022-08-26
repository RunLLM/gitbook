---
description: or, why MLOps set us down the wrong path
---

# The Aqueduct Philosophy

The last 10 years have seen an explosion in the tooling, interest, and applications of machine learning. One of the key driving forces behind the broad adoption of machine learning is access to a vibrant open-source ecosystem of software for preparing data and training models. From NumPy, Pandas, and Jupyter to SciKit-Learn, XGBoost, Pytorch, and Tensorflow, Python-based data science and machine learning software has made it easier than ever to build machine learning models.

Much of the innovation over the last 10 years has (rightly so) focused on processing and featurizing data, experimenting with different model architectures, and effectively training models. This is in line with our own experiences at UC Berkeley, where the data science curriculum we've developed ([Data 100](https://www.ds100.org)) teaches students the basics of SQL queries, exploratory data analysis, data preparation, modeling in SciKit-Learn and PyTorch, and model evaluation — all within the context of a Jupyter notebook.

Interestingly, while developing the Berkeley data science curriculum, we noticed that many of the students taking the data science courses were coming from disciplines outside of computer science and engineering. This new generation of highly data-literate students with backgrounds in humanities and physical sciences has the opportunity to drive the broad adoption of machine learning and transform the organizations they join. Yet, for them to succeed, they will need to overcome the significant engineering hurdles along the last mile of data science and machine learning: deploying, integrating, and supporting their models.

### The Missing Link

The part of the lifecycle that's received comparatively little attention in recent years is what happens once a model is built. Given the focus on building model to date, the missing link in generating value from machine learning is _enabling the_ _widespread use of models._

What we've observed by and large is that data teams rely on a medley of complicated, low-level cloud infrastructure tools — task orchestration systems like Airflow, low-level cloud infrastructure like Docker & Kubernetes, and hand-built connectors to data systems — to make predictions and share them with stakeholders. This set of tools is not only cumbersome and slow, it also provides very little visibility into errors and failures, which means that maintenance & debugging are painful.

The data scientists we've been working with have a combination of ML expertise and business knowledge that makes them uniquely poised to accelerate internal processes, inform business business decisions, and improve customer interactions at every company... if only the infrastructure could get out of their way.

### What is Production Data Science?

The problems we've heard from data scientists consistently come from one source — trying to force machine learning and data science workflows into infrastructure built for different use cases.\
We think this is an anti-pattern: Data scientists aren't software engineers, and making data scientists successful is going to require thinking tooling to meet them where they are. Rather than forcing data scientists to focus on complex cloud MLOps infrastructure, we need to build tools that allow data scientists to focus on solving business problems. Some of the companies we admire the most have done exactly this in other domains (e.g., Weights & Biases for ML experimentation, Vercel & Netlify for hosting web apps).

**Production data science (PDS) infrastructure enables data scientists to repeatably deliver high-quality predictions to their business without having to manage low-level cloud infrastructure tools.** PDS covers 3 core tasks:

* _Running data science in production (or just repeatedly)_**:** Rather than forcing you to learn, manage, and fight low-level tools like Docker, Kubernetes, or even Airflow, production data science infrastructure should enable you to run your code repeatably, wherever you’d like and with minimal configuration overhead.
* _Publishing predictions_: Once a data or ML pipeline is running, results can be shared with stakeholders & users; this generates business value and feedback, which can be turned into new, higher-quality data sets. Depending on the application of data science, predictions might need to be published as data, spreadsheets, visualizations, or endpoints — production data science infrastructure should support this diversity without added complexity.
* _Ensuring prediction quality_: Predictions can only be published if you have confidence in the results of your work, but data science projects can fail in subtle and unpredictable ways. Data teams need a clean, targeted way to measure and validate predictions (and input data), so you can be sure you’re publishing high quality data.

### Isn't this just MLOps?

We’ve written previously about [why existing MLOps tools are the wrong solution](https://blog.aqueducthq.com/posts/mlops-right-problem-wrong-solution), but… how is PDS different? The fundamental problem with MLOps is that the tools are overly focused on the specific problems faced by a few large tech companies. The problems faced by the data teams we’ve interviewed are not addressed by MLOps.

_First_, MLOps tools have primarily been built by and for engineers at the world’s biggest tech companies. As a consequence, they require armies of software engineers to configure, deploy, and manage. This is a highly ineffective use of time & resources, especially at smaller organizations. Production data science infrastructure is built to be lightweight and easy to manage, and critically, it abstracts away the underlying infrastructure rather than exposing it to data scientists.

_Second_, many applications of production data science don’t require software operations. Many data teams we’ve spoken to simply want to run a nightly workflow that publishes a dataset or generates a graph and emails it to the appropriate stakeholders. This doesn’t require complex, bespoke Kubernetes deployments — in fact, you can do this from a laptop! Production data science doesn't have to mean running a cluster of thousands of servers to serve single-digit millisecond latency predictions. Any data science that's being used to generate business value should be supported by production data science.

MLOps identified the right problem—providing data science-specific infrastructure—but it’s given us the wrong solutions. Data scientists need infrastructure built for their needs and skillsets; that’s what _production data science_ is all about.

### Why Aqueduct?

Aqueduct is the first production data science infrastructure.

The existing tools for deploying models are not designed with data scientists in mind—- they assume the user will casually build Docker containers, deploy Kubernetes clusters, and writes thousands of lines of YAML to deploy a single model. Data scientists are by and large not interested in doing that and there are better uses for their skills.

Aqueduct is designed and built for data scientists and meets the three core needs of production data science infrastructure:

* _Deploy_: Aqueduct has a simple Pythonic API that lets you define workflows in a few lines of code and run them anywhere from a laptop to a Kubernetes cluster.
* _Publish_: Aqueduct comes with a suite of connectors to common data systems and endpoints that allow you to publish predictions wherever they’re needed.
* _Monitor_: Aqueduct’s checks & metrics enable you — and your teammates — to ensure the correctness of predictions and measure them over time, enabling early detection of issues and quick bugfixes.

\
