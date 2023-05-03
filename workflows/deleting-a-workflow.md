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
from aqueduct import Client

client = Client()

demo_db = client.resource("aqueduct_demo")
reviews_table = demo_db.sql("select * from hotel_reviews;")

workflow_id = client.publish_flow(name="hotel_reviews", artifacts=[reviews_table]).id()  # Or set your workflow ID here.
client.delete_flow(workflow_id)
```

When deleting a workflow, you can additionally delete objects saved by the workflow. You can list the objects with the `list_saved_objects` method on the Flow object:

```python
from aqueduct import Client

client = Client()

demo_db = client.resource("aqueduct_demo")
reviews_table = demo_db.sql("select * from hotel_reviews;")

demo_db.save(reviews_table, table_name="reviews_table_2", update_mode="replace")
workflow_id = client.publish_flow(name="hotel_reviews", artifacts=[reviews_table]).id()  # Or set your workflow ID here.
flow = client.flow(workflow_id)
flow.list_saved_objects()
```

This returns a dictionary with the resource name as the key and the list of saved object names or paths as the values. For example, if the workflow wrote 3 tables, `table_1` and `table_2` into `aqueduct_demo` and `table_1` into the `postgres` resource, `flow.list_saved_objects()` would return:

```
{
    "aqueduct_demo": ["table_1", "table_2"]
    "postgres": ["table_1"]
}
```

You can pass this or any dictionary of the same format to the `delete_flow` method to delete the objects when deleting the flow. Tables can be published either in `append`, `replace`, or `fail` mode (see [Relational Databases](resources/data-systems/sql-systems/README.md) for details). When tables are saved in `append` mode, Aqueduct cannot guarantee that there was not data previously set by Aqueduct that's stored in the table. To prevent any unintentional deletion of data, please set `force=True` when deleting a data that was created in append mode.

The Aqueduct SDK will automatically prompt you to do this. It will not allow you to delete your workflow without the `force` flag.

```python
flow = client.flow(workflow_id)
all_objects = flow.list_saved_objects()
client.delete_flow(workflow_id, saved_objects_to_delete=all_objects, force=True)
```
