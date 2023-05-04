# Configuring your Python environment

Aqueduct works in any Python environment — Jupyter notebooks, Python scripts, or the IPython REPL. Simply run `import aqueduct`, and you're ready to go.

Within your Python environment, you can use Aqueduct's `global_config` to determine how Aqueduct executes the functions that you define. To set the global config, simply run:&#x20;

```python
import aqueduct
aqueduct.global_config({ "key": "value" })
```

You can configure the following parameters:&#x20;

* `lazy`: This determines whether Aqueduct will eagerly execute your functions or not. By default, `lazy` is `False`, meaning functions _will be eagerly executed_ — when an operator is called, it will immediately be run.  When `lazy` is set to `True`, functions will not be executed when they are invoked; instead, functions will only be executed when `.get()` is called on an Artifact.
  * Options: `True`, `False`
  * Default: `False`

{% hint style="info" %}
You may mix lazily and eagerly defined operators in the same flow. However, if an eager operator depends on a lazy operator, that lazy operator will ultimately have to be executed, and will gain all the type enforcement benefits of an eagerly executed operator!
{% endhint %}

* `engine`: This determines where Aqueduct will execute your code. By default, the engine is set to `None`, meaning the default Aqueduct execution engine will be used. When you set a particular system here, all code execution — both for _ad hoc_ execution and for workflows published from this environment — will use this system. &#x20;
  * Options: any of the [compute-systems](../resources/compute-systems/ "mention") you've connected Aqueduct to
  * Default: `None`, indicating that code will be run using the default Aqueduct execution engine
