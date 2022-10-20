# Debugging a Failed Workflow

Despite our best efforts, our prediction pipelines often fail. To help avoid going on wild goose chases, Aqueduct gives you significant insight into understanding when and where workflows failed and resolving things quickly.&#x20;

### Notifications: Knowing a Workflow Failed

Half the battle is often knowing a workflow failed. Aqueduct sends notifications on the success or failure of every workflow execution, which you can see under the notifications tab on the top-right side of the UI.

&#x20;

![Aqueduct's Notifications pane](<../.gitbook/assets/notifications_failed.png>)

### Workflow Status: Finding Your Errors

The workflow overview page shows a status for the current run. Additionally, the DAG visualization allows you to narrow down the errors to certain operators -- nodes that are green executed successfully, nodes that are yellow had warnings, nodes that are red failed or had errors, and nodes that are grey didn't execute (because an upstream operator failed or errored).&#x20;

![Failed Workflow Overview](<../.gitbook/assets/failed_workflow_overview.png>)

In order to find out what went wrong, you can consult the workflow status bar, which shows a condensed overview of all the errors, warnings, logs, and successful operators associated with this workflow run.

![The Workflow Status Bar gives a condensed overview of the workflow run](<../.gitbook/assets/failed_workflow_status_bar.png>)

If more context is needed around a failure, you can click on the node that failed, which will cause a pane to pop up from the right. This pane will show additional context, such as the code itself, its inputs and outputs, etc.

![The Node View gives more detailed context around a failure](<../.gitbook/assets/failed_node.png>) 

### Debugging Your Data



We're working on adding support for retrieving the code and data executed during a particular workflow run, so you can recreate and debug errors locally -- stay tuned!
