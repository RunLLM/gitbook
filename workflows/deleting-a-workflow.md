# Deleting a Workflow

Deleting a workflow can be done from both the UI and SDK. Deleting a workflow will delete the following:

* The code associated with the workflow.
* The snapshots of the intermediary data (see [workflow-versions.md](workflow-versions.md "mention")).&#x20;
* All the metadata associated with the workflow.

#### Aqueduct UI

On the Aqueduct UI, navigate to the settings pane by clicking on the gear icon on the top right. At the bottom of the settings pane, you will see the option to delete the workflow. This will prompt you to confirm the workflow deletion.&#x20;

![](<../.gitbook/assets/image (6) (1).png>)

After you press the "Delete" button, you will be asked to confirm you wish to delete the workflow. You will also be given a list of objects saved by the workflow. You can select the objects in the list for deletion. Any objects you pick will be completely deleted.

![](<../.gitbook/assets/workflow_deletion_confirmation_page.png>)

Once you confirm deletion of the workflow, Aqueduct will delete the workflow's metadata as well as any of the data objects you selected on the previous screen. Object deletion is done best-effort, and the workflow will be deleted whether or not the associated objects are successfully deleted. After workflow deletion has completed, you will be shown a list of all the objects that Aqueduct attempted to delete and whether those delete operations succeeded or not.

![](<../.gitbook/assets/object_deletion_results_page.png>)

#### Aqueduct SDK

From the SDK, you can use the `delete_flow` method on the Aqueduct client and pass in the workflow's UUID in order to delete a workflow:

```python
workflow_id = "0c007eff-6ae0-4a1a-a114-5f16164ffcdf" # Set your workflow ID here.
client.delete_flow(workflow_id)
```

