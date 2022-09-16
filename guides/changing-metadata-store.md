# Changing Metadata Store

Aqueduct uses a storage layer for tracking workflow, operator, and artifact metadata. The default storage layer is the local filesystem. Aqueduct currently also supports using the following services for its metadata store:
- AWS S3
- Google Cloud Storage (GCS)

There are two ways to change the metadata store. You can either use the `aqueduct` CLI or connect a new storage integration from the UI.

## Using the Aqueduct CLI
The `aqueduct storage` command can be used to change the metadata store. It currently supports changing between the local filesystem and AWS S3. (Support for changing to GCS will be added soon.)

In order to change the store to an AWS S3 bucket, you must use the following command:
```
aqueduct storage --use s3 --path <YOUR_S3_BUCKET_PATH>
[--region <YOUR_S3_BUCKET_REGION>] 
[--credentials <YOUR_AWS_CREDENTIALS_FILEPATH>]
[--profile <YOUR_AWS_CREDENTIALS_PROFILE_NAME>]
```

After you have done this, you need to restart the Aqueduct server.

## Using the Aqueduct UI
The Aqueduct UI can be used to change the metadata store to AWS S3 or GCS. You
do this by connecting a new integration of the respective service. As part of the
integration connection dialog, you must select the box at the bottom that says
`Use this integration for Aqueduct metadata storage.`.

![](<../.gitbook/assets/change_metadata_store.png>)