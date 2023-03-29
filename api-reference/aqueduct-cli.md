# Aqueduct CLI

This page provide a detailed walkthrough of the Aqueduct CLI.&#x20;

* ``[`start`](aqueduct-cli.md#undefined)``
* ``[`version`](aqueduct-cli.md#version)``
* ``[`install`](aqueduct-cli.md#install)``
* ``[`apikey`](aqueduct-cli.md#apikey)``
* ``[`clear`](aqueduct-cli.md#undefined)``

#### start

`aqueduct start` starts the Aqueduct server, which includes both the Aqueduct API layer as well as the Aqueduct UI. If this is your first time starting Aqueduct or if you've recently upgraded to a newer version of Aqueduct, we will download the latest versions of the backend and UI code before starting the server.

`start` has the following optional arguments:

* `--verbose`: by default, all `INFO` and `DEBUG` logs are written to a log file; if run in verbose mode, Aqueduct will also print all log statements on `stdout`.
* `--expose`: by default, Aqueduct is only accessible from the machine on which it is being run; if run with `--expose`, the server and UI will be accessible from other machines.
* `--port`: by default, Aqueduct runs on port 8080; with `--port`, you can configure which port you would like the Aqueduct server to listen on.

`aqueduct start` is a blocking command, meaning that it will start the server in a process that takes over your terminal window.

#### version

`aqueduct version` displays which version of Aqueduct you are running locally. For more details on updating Aqueduct, see [updating-aqueduct.md](../installation-and-configuration/updating-aqueduct.md "mention").

#### install

`aqueduct install <connector>` installs the dependencies required for `<connector>` on your machine. In most cases, these are `pip` packages on a system-by-system basis, but certain connectors (MySQL & Microsoft SQL Server) require special configuration -- see [Broken link](broken-reference "mention") for more details.

#### apikey

`aqueduct apikey` reports the API key associated with the server running on this instance.

#### clear

`aqueduct clear` removes the local Aqueduct installation by deleting the server, UI code, and any associated metadata.

#### storage

`aqueduct storage` allows you to configure Aqueduct's artifact storage. Using Aqueduct storage, you can either set your artifact storage to be the local filesystem or an S3 bucket.&#x20;

To set the artifact store to a path on the local file system, use `aqueduct storage --use local`:

```bash
AQUEDUCT_STORAGE_PATH=$HOME/.aqueduct
aqueduct storage --use local --path $AQUEDUCT_STORAGE_PATH
```

To set the artifact store to an S3 bucket, use `aqueduct storage --use s3`:

```bash
S3_BUCKET=s3://aqueduct_bucket/path/to/storage
AWS_CREDENTIALS=$HOME/.aws/credentials
AWS_CREDENTIALS_PROFILE=default

aqueduct storage \
    --use s3 \
    --path $S3_BUCKET \
    --credentials $AWS_CREDENTIALS \
    --profile $AWS_CREDENTIALS_PROFILE
```
