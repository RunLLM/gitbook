# Usage Stats Collection

As of v0.1.9, Aqueduct collects usage statistics, which helps us identify bugs and better understand
how to improve our user experience. Usage stats collection is enabled by default, and can be disabled
by adding `--disable-usage-stats` to `aqueduct start`.

Please refer to our [usage stats class](https://github.com/aqueducthq/aqueduct/blob/main/src/golang/cmd/server/middleware/usage/models.go) to see what data we collect.
It is important to note that we do not collect any sensitive or personally identifiable data. We securely
hash your machine ID so that no information related to the original ID will be revealed. We also redact
any UUIDs related to your workflows, operators, artifacts, etc.