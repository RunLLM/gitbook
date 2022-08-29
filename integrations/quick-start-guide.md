# Integration Management Quickstart Guide
In this quickstart guide, we will go over key features for integration management.

## Integrations Page
The integrations page is your home for adding and viewing integrations connected to your Aqueduct account. You can naviate to integrations page by clicking `Integrations` in the left menu bar. This page has two main sections: [add an integration](#add-an-integration) and [connected integrations](#connected-integrations).

### Add an Integration
To add an integration, simply click the icon for the integration you would like to connect. You would need to provide your service details together with your credentials. You could find more details [here](./adding-an-integration/) on connecting to s3, bigquery, and Google Cloud Storage (GCS).

Once you added an integration, you can refer to [using integrations guide](./using-integrations/) to use the integration in Aqueduct SDK.

### Connected Integrations
You can view a list of all connected integrations. You can click into each item to view more details or manage that integration. Every Aqueduct server comes with a [demo integration](./aqueduct-demo-integration.md) that you can play around with.

## Integration Detail Page
Integration Detail Page is the console for you to manage a single integration. You can navigate to this page by clicking an item in [Connected Integrations](#connected-integrations) from [Integrations Page](#integrations-page) you can view details about an integration and make changes to it:
* [Integration Information](#integration-information)
* [Test-Connecting an Integration](#test-connecting-an-integration)
* [Editing an Integration](#editing-an-integration)
* [Deleting an Integration](#deleting-an-integration)
* [Viewing Data in an Integration](#viewing-data-in-an-integration)
* [Viewing Workflows Using an Integration](#viewing-workflows-using-an-integration)

### Integration Information
At top of the integration detail page. You can view basic information about this integration. There are also a number of options to manage it, including [test-connection](#test-connecting-an-integration), [edit the integration](#editing-an-integration), and [delete the integration](#deleting-an-integration).
![](<../.gitbook/assets/integration_information.png>)

### Test-Connecting an Integration
In `Options` menu, you can test-connect an integration to verify if it's currently available. This is especially helpful to confirm whether your credentials are expired.

### Editing an Integration
In `Options` menu, you can edit an integration to update its name and credentials. The changes will also update all existing workflows using this integration.

We do not support updating fields that requires potential data migration. For example, `host` and `database` for relational databases. To make changes on these fields, you would need to follow a migration process:
1. Create a new integration.
2. Perform any necessary data migration in your integration.
3. [Update your workflows](../workflows/editing-a-workflow.md) to use the new integration.

### Deleting an Integration
If you don't have any workflow using an integration, you can delete that integration from `Options` menu. Otherwise, you would need to [delete all workflows using this integration](../workflows/deleting-a-workflow.md) first. You can refer to [workflows section](#viewing-workflows-using-an-integration) to find all workflows using this integration.

### Viewing Data in an Integration
For **relational databases**, in `Preview` section, you can view tables stored in this integration. You can also get a preview of each table by typing the table name in the search box. You can find more details [here](./viewing-data-in-an-integration.md) on viewing data in integration.
![](<../.gitbook/assets/integration_data.png>)

### Viewing Workflows Using an Integration
`Workflows` section gives you an overview of all data extracted from and saved to this integration. You can view a list of workflows using this integration. For each workflow, you can view details about all operators on this integration, including those from previous versions. It is especially useful to verify all workflows depending on this integration before you [making changes](#editing-an-integration) or [deleting it](#deleting-an-integration).
![](<../.gitbook/assets/integration_workflows.png>)
