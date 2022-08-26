In this quick start guide, we will go over key features for integration management.

# Integrations Page
You can naviate to integrations page by clicking `Integrations` in the left menu bar. This page has two main sections: [add an integration](#add-an-integration "mention") and [connected integrations](#connected-integrations "mention").

## Add an Integration
To add an integration, simply click the icon for the integration you would like to connect. You would need to provide your service details together with your credentials. You could find more details [here](./adding-an-integration/) on connecting to s3, bigquery, and Google Cloud Storage (GCS).

Once you added an integration, you can refer to [using integrations guide](./using-integrations/) to use the integration in Aqueduct SDK.

## Connected Integrations
You can view a list of all connected integrations. You can click into each item to view more details or manage that integration. Every Aqueduct server comes with a [demo integration](./aqueduct-demo-integration.md) that you can play around with.

# Integration Detail Page
In this page, you can view details about an integration and make changes to it:
* [Integration Information](#integration-information "mention")
* [Test-Connecting an Integration](#test-connecting-an-integration "mention")
* [Editing an Integration](#editing-an-integration "mention")
* [Deleting an Integration](#deleting-an-integration "mention")
* [Viewing Data in an Integration](#viewing-data-in-an-integration "mention")
* [Viewing Workflows Using an Integration](#viewing-workflows-using-an-integration "mention")

## Integration Information
At top of the integration detail page. You can view basic information about this integration. There are also a number of options to manage it, including [test-connection](#test-connecting-an-integration "mention"), [edit the integration](#editing-an-integration "mention"), and [delete the integration](#deleting-an-integration "mention").
![](<../.gitbook/assets/integration_information.png>)

## Test-Connecting an Integration
In `Options` menu, you can test-connect an integration to verify if it's currently available. This is especially helpful to confirm whether your credentials are expired.

## Editing an Integration
In `Options` menu, you can edit an integration to update its name and credentials. The changes will also update all existing workflows using this integration.

For now, we do not support updating fields that requires potential data migration. For example, `host` and `database` for relational databases. To make changes on these fields, you would need to follow a migration process:
1. Create a new integration.
2. Perform any necessary data migration in your integration.
3. [Update your workflows](../workflows/editing-a-workflow.md) to use the new integration.

## Deleting an Integration
If you don't have any workflow using an integration, you can delete that integration from `Options` menu. Otherwise, you would need to [delete all workflows using this integration](../workflows/deleting-a-workflow.md) first.

## Viewing Data in an Integration
For **relational databases**, in `Preview` section, you can view tables stored in this integration. You can also get a preview of each table by typing the table name in the search box. You can find more details [here](./viewing-data-in-an-integration.md) on viewing data in integration.
![](<../.gitbook/assets/integration_data.png>)

## Viewing Workflows Using an Integration
In `Workflows` section, you can view a list of workflows using this integration together with all operators using the integration on each workflow.
![](<../.gitbook/assets/integration_workflows.png>)
