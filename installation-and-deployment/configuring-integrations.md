# Configuring Integrations

To install the dependencies required to connect to any integration, please run `aqueduct install [integration_name]`. For example, to enable connections to Postgres, run `aqueduct install postgres`. For most integrations, this command will simply install the required package dependencies necessary to connect to that system.

However, MySQL and Microsoft SQLServer have special requirements that we describe below.

### Configuring MySQL

In order to connect to MySQL from your Aqueduct server, you will need to run the following command:

```bash
aqueduct install mysql
```

When you run this command, the Aqueduct CLI will take the following steps:

* On Linux operating systems, we install the following packages using the default system package manager (`apt-get` or `yum`):
  * the Python3 development headers
  * the MySQL client
* On macOS, we install the following package using `brew` -- if you do not have [Homebrew](https://brew.sh/) installed, please install it before attempting to configure this connector:
  * the MySQL client (`brew install mysql`)

### Configuring Microsoft SQLServer

In order to connect to MySQL from your Aqueduct server, you will need to run the following command:

```bash
aqueduct install sqlserver
```

When you run this command, the Aqueduct CLI will take the following steps:

* On Linux operating systems, we install the following packages using the default system package manager (`apt-get` or `yum`):
  * adds the requisite packages via the local package manger and installs the `msodbcsql17` and `mssql-tools` packages as described [here](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver16)
* On macOS, we install the following package using `brew` -- if you do not have [Homebrew](https://brew.sh/) installed, please install it before attempting to configure this connector:
  * adds the [Microsoft MSSQL release repository](https://github.com/Microsoft/homebrew-mssql-release) as a brew tap and installs the `msodbcsql17` and `mssql-tools` packages using the new tap (as specified [here](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver16))[https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver16](https://docs.microsoft.com/en-us/sql/connect/odbc/linux-mac/install-microsoft-odbc-driver-sql-server-macos?view=sql-server-ver16)
