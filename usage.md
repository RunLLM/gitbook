# Usage Stats Collection

As of v0.1.9, Aqueduct collects usage statistics, which help us identify bugs and better understand
how to improve our user experience. The server start logs will notify you that usage information is
being collected by default. You can disable it by running the server with the following flag:

```bash
aqueduct start --disable-usage-stats
```

It is important to note that we do not collect any sensitive or personally identifiable data. We securely
[hash your machine ID](https://github.com/denisbrodbeck/machineid) so that no information related to the original ID will be revealed. We also redact
any UUIDs related to your workflows, operators, artifacts, etc.

Currently, we collect stats such as the endpoints accessed, the timestamp and latency of the requests, and
the HTTP status code of the responses.
Please refer to our [usage stats class](https://github.com/aqueducthq/aqueduct/blob/main/src/golang/cmd/server/middleware/usage/models.go) to see the full list of stats we collect.