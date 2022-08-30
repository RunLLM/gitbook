# Integration Management Guide
In this guide, we will go over key features for integration management.

## Integrations Page
The integrations page is your home for adding and viewing integrations connected to Aqueduct. You can naviate to integrations page by clicking on `Integrations` in the left menu bar. This page has two main sections: [add an integration](#adding-an-integration) and [connected integrations](#connected-integrations).

### Adding an Integration
To add an integration, simply click the icon for the integration you would like to connect. You will need to provide your service details together with your credentials. You can find more details [here](./adding-an-integration/) on connecting to AWS S3, Google Bigquery, and Google Cloud Storage (GCS).

Once you have added an integration, you can refer to [using integrations guide](./using-integrations/) to use the integration in Aqueduct SDK.

### Connected Integrations
Aqueduct displays a list of all connected integrations. You can click into each item to view more details, or make changes to that integration. Every Aqueduct server comes with a [demo integration](./aqueduct-demo-integration.md) with preloaded sample datasets.

## Integration Details Page
The Integration Detail Page is the console for you to manage a single integration. You can navigate to this page by clicking on an item in [Connected Integrations](#connected-integrations) from [Integrations Page](#integrations-page). This page displays various information about this integration and also allows you to make changes to it:
* [Integration Information](#integration-information)
* [Test Your Integration Connection](#test-your-integration-connection)
* [Editing an Integration](#editing-an-integration)
* [Deleting an Integration](#deleting-an-integration)
* [Viewing Data in an Integration](#viewing-data-in-an-integration)
* [Viewing Workflows Using an Integration](#viewing-workflows-using-an-integration)

### Integration Information
At top of the integration detail page. You can view basic information about this integration. The `Options` menu allows you to [test your integration connection](#test-your-integration-connection), [edit the integration](#editing-an-integration), or [delete the integration](#deleting-an-integration).
![](<../.gitbook/assets/integration_information.png>)

### Test Your Integration Connection
In the `Options` menu, you can test-connect an integration to verify if it's currently available. This is especially helpful to confirm whether your credentials are expired.

### Editing an Integration
In the `Options` menu, you can edit an integration to update its name and credentials. The changes will propengate to all existing workflows using this integration. Before you make changes, we recommend checking the [workflows section](#viewing-workflows-using-an-integration) to verify that all workflows using this integration will continue to work properly.

Certain fields are protected from editing on a per-integration basis. For example, `host` and `database` fields for relational databases. To make changes on these fields, you would need to follow a migration process:
1. Create a new integration.
2. Perform any necessary data migration in your integration.
3. [Update your workflows](../workflows/editing-a-workflow.md) to use the new integration.

### Deleting an Integration
If you don't have any workflow using an integration, you can delete that integration from the `Options` menu. Otherwise, you will need to [delete all workflows using this integration](../workflows/deleting-a-workflow.md) first. You can refer to the [workflows section](#viewing-workflows-using-an-integration) to see a list of all workflows using this integration.

### Viewing Data in an Integration
For **relational databases**, in the `Preview` section, you can view tables stored in this integration. You can also get a preview of each table by typing the table name in the dropdown menu. You can find more details [here](./viewing-data-in-an-integration.md) on viewing data in integration.
![](<../.gitbook/assets/integration_data.png>)

### Viewing Workflows Using an Integration
The `Workflows` section gives you an overview of all data extracted from and saved to this integration. You can view a list of workflows using this integration. For each workflow, you can view details about all operators on this integration, including those from previous versions.

This section is particularly useful to verify depending workflows before you make any changes. Before you [edit this integration](#editing-an-integration), we recommend you verify that all the workflow depending on this integration will continue to be correct. Before [deleting this integration](#deleting-an-integration), you will need to verify that no workflow is using this integration .
![](<../.gitbook/assets/integration_workflows.png>)
