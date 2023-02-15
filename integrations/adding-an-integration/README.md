# Adding an Integration

New integrations can be connected from the Aqueduct UI on the integrations page, which can be accessed from the menu bar on the left.

<figure><img src="../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

Every integration has a slightly different set of requirements. When you click on the icon for each integration, you'll be asked to provide a unique name for that integration, the host endpoint for the server running the system, as well as the requisite authentication credentials.

Several systems -- [AWS S3](connecting-to-aws-s3.md), [AWS Lambda](connecting-to-aws-lambda.md), [Google BigQuery](connecting-to-google-bigquery.md), [Kubernetes](connecting-to-k8s-cluster.md), [Databricks on AWS](connecting-to-databricks.md) and [conda](connecting-to-conda.md) -- require special configuration parameters, which we'll explain next.

Once an integration has been added to Aqueduct, you can access it from the Python SDK by calling `client.integration(integration_name)`.
