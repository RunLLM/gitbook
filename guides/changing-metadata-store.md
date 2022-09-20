# Changing Metadata Store

Aqueduct uses a storage layer for tracking workflow, operator, and artifact metadata. The default storage layer is the local filesystem. Aqueduct currently also supports using the following services for its metadata store:
- AWS S3
- Google Cloud Storage (GCS)

The Aqueduct UI should be used to change the metadata store to AWS S3 or GCS. You
do this by connecting a new integration of the respective service. As part of the
integration connection dialog, you must select the box at the bottom that says
`Use this integration for Aqueduct metadata storage.`.

![](<../.gitbook/assets/change_metadata_store.png>)

We will add support shortly for switching back to using the local filesyste.