<!---
Note: This tutorial is meant for Google Cloud Shell, and can be opened by going to
http://gstatic.com/cloudssh/images/open-btn.svg)](https://console.cloud.google.com/cloudshell/open?git_repo=https://github.com/mesmacosta/sqlserver-to-datacatalog-tutorial&tutorial=TUTORIAL.md)--->
# SQLServer to Data Catalog Tutorial

<!-- TODO: analytics id? -->
<walkthrough-author name="mesmacosta@gmail.com" tutorialName="SQLServer to Data Catalog Tutorial" repositoryUrl="https://github.com/mesmacosta/sqlserver-to-datacatalog-tutorial"></walkthrough-author>

## Intro

This tutorial will walk you through the execution of the SQLServer to Data Catalog Tutorial.

## Environment

Let's start by setting up your environment.

## Set Up your Project

Start by setting your project ID. Replace the placeholder to your project.
```bash
gcloud config set project MY_PROJECT_PLACEHOLDER
```

Next load it in a environment variable.
```bash
export PROJECT_ID=$(gcloud config get-value project)
```

## Create the SQLServer Database

You can use the open source scripts at [Cloud SQL SQLServer Tooling](https://github.com/mesmacosta/cloudsql-sqlserver-tooling) to create and populate your SQLServer instance.

Clone the github repository:
```bash
git clone https://github.com/mesmacosta/cloudsql-sqlserver-tooling
```
Go to the cloned repo directory:
```bash
cd cloudsql-sqlserver-tooling
```

Next execute the `init-db.sh` script.
This will create your SQLServer instance and populate it with random schema.
```bash
source init-db.sh
```

## Set Up the Service Account

Create a Service Account.
```bash
gcloud iam service-accounts create sqlserver2dc-credentials \
--display-name  "Service Account for SQLServer to Data Catalog connector" \
--project $PROJECT_ID
```

Next create and download the Service Account Key.
```bash
gcloud iam service-accounts keys create "sqlserver2dc-credentials.json" \
--iam-account "sqlserver2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" 
```

Next add Data Catalog admin role to the Service Account.
```bash
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member "serviceAccount:sqlserver2dc-credentials@$PROJECT_ID.iam.gserviceaccount.com" \
--quiet \
--project $PROJECT_ID \
--role "roles/datacatalog.admin"
```

## Execute the SQLServer connector

You can build the SQLServer connector yourself by going to
[this GitHub repository](https://github.com/GoogleCloudPlatform/datacatalog-connectors-rdbms/tree/master/sqlserver2datacatalog).

To facilitate its usage, we are going to use a docker image. 
The environment variables were loaded by the `init-db.sh` script.

Execute the connector:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/sqlserver2datacatalog:stable \
--datacatalog-project-id=$PROJECT_ID \
--datacatalog-location-id=us-central1 \
--sqlserver-host=$public_ip_address \
--sqlserver-user=$username \
--sqlserver-pass=$password \
--sqlserver-database=$database
```

## Check the results of the script

After the script finishes, you can go to Go to Data Catalog
[Search UI](https://console.cloud.google.com/datacatalog?q=system=sqlserver)
 and search for SQLServer metadata.

## Cleaning up

The easiest way to avoid incurring charges to your Google Cloud account for the resources used in this tutorial is to delete 
the project you created. Otherwise you can run the clean up scripts below.

**To delete the project**, follow the steps below:

1.  In the Cloud Console, [go to the Projects page](https://console.cloud.google.com/iam-admin/projects).

2.  In the project list, select the project that you want to delete and click **Delete project**.

    ![N|Solid](https://storage.googleapis.com/gcp-community/tutorials/partial-redaction-with-dlp-and-gcf/img_delete_project.png)
    
3.  In the dialog, type the project ID, and then click **Shut down** to delete the project.

**To delete the created resources** and mantain the project,
 follow the steps below:

Delete the SQLServer metadata:
```bash
./cleanup-db.sh
```

Execute the cleaner container:
```bash
docker run --rm --tty -v \
"$PWD":/data mesmacosta/sqlserver-datacatalog-cleaner:stable \
--datacatalog-project-ids=$PROJECT_ID \
--rdbms-type=sqlserver \
--table-container-type=schema
```

Delete the SQLServer database:
```bash
./delete-db.sh
```
## Search for the SQLServer Entries

Go to Data Catalog search UI:
[Search UI](https://console.cloud.google.com/datacatalog?q=system=sqlserver)

Check the search results, and verify that there are no results. Entries for the system `sqlserver` 
have been deleted.

## Congratulations!

<walkthrough-conclusion-trophy></walkthrough-conclusion-trophy>

You've successfully finished the SQLServer to Data Catalog Tutorial.
