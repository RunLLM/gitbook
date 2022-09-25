# Connecting to Google Cloud Storage

Aqueduct connects to Google Cloud Storage using a [`ServiceAccount`](https://cloud.google.com/docs/authentication#service-accounts). This guide will walk you through creating a `ServiceAccount`, attaching the necessary roles to it so that Aqueduct can access your Google Cloud Storage bucket, and downloading the JSON credentials file that is required by the Aqueduct UI.

1. Follow Google Cloud's documentation [here](https://cloud.google.com/iam/docs/creating-managing-service-accounts#iam-service-accounts-create-console) for creating a `ServiceAccount`.
2. Navigate to the bucket you want to connect and go to the `Permissions` tab.&#x20;
3. Attach a `Storage Object Admin` role to the `ServiceAccount` you created in step 1. &#x20;
4. Follow the instructions [here](https://cloud.google.com/iam/docs/creating-managing-service-account-keys) for creating a key for the `ServiceAccount` you created in step 1. Once you download the service account key file, you can use it when connecting to Google Cloud Storage on the Aqueduct UI.
