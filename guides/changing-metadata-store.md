# Changing Metadata Store

Aqueduct uses a storage layer for tracking workflow, operator, and artifact metadata. It stores execution information and artifact content. The default storage layer is the local filesystem. Aqueduct currently also supports using the following services for its metadata store:
- AWS S3
- Google Cloud Storage (GCS)

You may want to change the metadata store to avoid consuming too much space on your local filesystem. Aqueduct requires you to change the
metadata store when using a different engine to run workflows:
- Airflow requires S3
- Kubernetes requires either S3 or GCS
- AWS Lambda requires S3

The Aqueduct UI should be used to change the metadata store to AWS S3 or GCS. You
do this by connecting a new integration of the respective service. As part of the
integration connection dialog, you must select the box at the bottom that says
`Use this integration for Aqueduct metadata storage.`.

![](<../.gitbook/assets/change_metadata_store.png>)

We do not currently support switching back to the local filesystem, but are working on this.