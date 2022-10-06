# Updating Aqueduct

We typically cut weekly patch releases for Aqueduct with feature releases, bugfixes, and general improvements. Every release gets published as an update to our `pip` package, `aqueduct-ml`.&#x20;

Updating Aqueduct is simple.&#x20;

1. Stop your existing Aqueduct server. Wherever you're running Aqueduct, you can stop the server with `ctrl-c`. If you do not stop the server, the package update will not take effect.
2. Run `pip3 install --upgrade aqueduct-ml`. This will download the latest version of the Aqueduct package.&#x20;
3. Run `aqueduct start` again. This will download the latest Aqueduct binaries restart your server.

That's it! You're ready to go with the latest version of Aqueduct.

### Update Guidelines

Patch releases will never remove functionality, though we will mark features that will be removed in future releases as deprecated. Deprecated functionality will be removed two minor releases after the current release. For example, if we mark an API call as deprecated in `v0.1.2`, it will be removed in `v0.3`.

While we aren't mature enough yet to be publishing major version releases, we will publish guidelines for major version upgrades, which might include future breaking API changes, here in the future.
