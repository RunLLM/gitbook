# Deleting a Workflow

Deleting a workflow can be done from both the UI and SDK. Deleting a workflow will delete the following:

* The code associated with the workflow.
* The snapshots of the intermediary data (see [workflow-versions.md](workflow-versions.md "mention")).&#x20;
* All the metadata associated with the workflow.

#### Aqueduct UI

On the Aqueduct UI, navigate to the settings pane by clicking on the gear icon on the top right. At the bottom of the settings pane, you will see the option to delete the workflow. This will prompt you to confirm the workflow deletion.&#x20;

![](<../.gitbook/assets/image (6) (1).png>)

After workflow deletion has succeeded, you will be redirected to the workflows page.

#### Aqueduct SDK

From the SDK, you can use the `delete_flow` method on the Aqueduct client and pass in the workflow's UUID in order to delete a workflow:

```python
workflow_id = "0c007eff-6ae0-4a1a-a114-5f16164ffcdf" # Set your workflow ID here.
client.delete_flow(workflow_id)
```

When deleting a workflow, you can additionally delete objects saved by the workflow. You can list the objects with the `list_saved_objects` method on the Flow object:

```python
workflow_id = "0c007eff-6ae0-4a1a-a114-5f16164ffcdf" # Set your workflow ID here.
flow = client.flow(workflow_id)
flow.list_saved_objects()
```

This returns a dictionary with the integration name as the key and the list of saved object names or paths as the values. You can pass this or any dictionary of the same format to the `delete_flow` method to delete the objects when deleting the flow. If any of them were saved with update mode append, you will additionally need to set `force=True` to confirm you understand the entire object would be deleted, otherwise the whole workflow deletion will fail.

```python
workflow_id = "0c007eff-6ae0-4a1a-a114-5f16164ffcdf" # Set your workflow ID here.
flow = client.flow(workflow_id)
all_objects = flow.list_saved_objects()
client.delete_flow(workflow_id, saved_objects_to_delete=all_objects, force=True)
```
