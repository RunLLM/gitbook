# Scheduling Workflwos

Workflows can be executed on a set schedule, triggered on the completion of another workflows, or triggered manually.&#x20;

## Schedules

A workflow's schedule can either be set from the Python SDK when calling `publish_flow` (see [creating-a-workflow.md](creating-a-workflow.md "mention")) or from the web UI.

### Aqueduct SDK

The Aqueduct SDK exposes four helper functions `hourly`, `daily`, `weekly`, and `monthly`, which each allow you to create a schedule of the corresponding frequency. The resulting schedule can be passed into the `schedule` argument of the `publish_flow` call.

```python
import hourly, daily, weekly, monthly, DayOfWeek from aqueduct

# Execute at the 10th minute of every hour
hourly_schedule = hourly(minute=10) 

# Execute at 16:00 (or 4:00pm) every day.
daily_schedule = daily(hour=16, minute=0)

# Execute at 08:00 (or 8:00am) every Monday.
weekly_schedule = weekly(day=DayOfWeek.Monday, hour=8, minute=0)

# Execute at 11:30 (or 11:30am) on the 3rd day of every month.
monthly_schedule = monthly(day=3, hour=11, minute=30)
```

### Aqueduct UI

On the Aqueduct UI, navigate to the page for any given workflow and click on the settings tab. Under the schedule section, you can select whether the workflow should update hourly, daily, weekly, or monthly. For each selection, you'll see the corresponding configuration -- for example, if you select weekly, you'll be asked to select the day of the week and the time on that day.&#x20;

<figure><img src="../.gitbook/assets/image (19).png" alt=""><figcaption></figcaption></figure>

Once you've made your selection, you can hit `Save` to finalize your changes.

## Cascading Workflows

Workflows can be scheduled to execute upon the completion of another workflow.

If a source workflow is provided, the target workflow will be triggered at the end of each successful run of the source workflow. If the source workflow does not complete successfully, the target workflow will not be triggered. Each workflow can have at most one source workflow. Aqueduct will return an error (on the SDK and UI) to prevent you from forming a cycle amongst cascading workflow.

### Defining Source Workflow from the SDK

The source workflow can be set when publishing a workflow from the SDK. The complete documentation for publishing a workflow from the SDK can be found [here](creating-a-workflow.md#publishing-a-workflow).

The `source_flow` argument in the following code snippet allows you to set the source workflow:

```python
workflow_a = client.flow('8fb25dc4-62ed-44a3-872d-c3ff988c8dd3')

# source_flow can be a Flow object, workflow name, or workflow ID
flow = client.publish_flow(name='workflow_b', 
                           artifacts=[data],
                           source_flow=source_flow)
```

### Defining Source Workflow from the UI

The source workflow can be modified from the UI from the workflow settings tab.

## Manual Triggers

Workflows can always be manually triggered (both from the UI and SDK), whether or not they have a set schedule.

### Aqueduct SDK

From the SDK, you can use the `trigger` method on the Aqueduct Client and pass in the workflow's UUID to trigger a workflow:

```python
workflow_id = "0c007eff-6ae0-4a1a-a114-5f16164ffcdf" # Set your workflow ID here.
client.trigger(id=workflow_id)
```

The workflow's ID is printed out when the workflow is created and can also be found in the URL For the workflows page. **NOTE**: We will soon be adding the workflow ID to the settings tab of the UI.

### Aqueduct UI

On the Aqueduct UI, navigate to the page for your workflow. To the right of the workflow DAG visualization, you will see a play button titled `Run Workflow`. Once you click on it, you will be prompted you to confirm a new workflow run and enter optional values for any parameters.

<figure><img src="../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

