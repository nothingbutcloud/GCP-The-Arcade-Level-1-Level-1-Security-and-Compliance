# [ARC129 - Secure BigLake Data: Challenge Lab](https://www.cloudskillsboost.google/games/5058/labs/33036)

## Overview

In a challenge lab you’re given a scenario and a set of tasks. Instead of following step-by-step instructions, you will use the skills learned from the labs in the course to figure out how to complete the tasks on your own! An automated scoring system (shown on this page) will provide feedback on whether you have completed your tasks correctly.

When you take a challenge lab, you will not be taught new Google Cloud concepts. You are expected to extend your learned skills, like changing default values and reading and researching error messages to fix your own mistakes.

To score 100% you must successfully complete all tasks within the time period!

## Challenge scenario

You are just starting your junior data engineer role. So far you have been helping teams create and manage BigLake assets.

You are expected to have the skills and knowledge for these tasks.

### Your challenge

You are asked to help a newly formed development team with some of their initial work on a new project. Specifically, they need a new BigLake table from a Cloud Storage file with the appropriate permissions to limit access to sensitive data columns; you receive the following request to complete the following tasks:

- Create a BigLake table from an existing file on Cloud Storage.
- Apply and verify policy tags to restrict access to columns containing sensitive data.
- Remove direct IAM permissions to Cloud Storage for other users (after policy tags have been applied to protect the data).

Some standards you should follow:

- Ensure that any needed APIs (such as Data Catalog and BigQuery Connection API) are successfully enabled and that necessary service accounts have the appropriate permissions.
- Create all resources in the multiple regions in United States, unless otherwise directed.

Each task is described in detail below, good luck!

## Solution

Project ID: `qwiklabs-gcp-04-246c58bc3b91`


### Task 1. Create a BigLake table using a Cloud Resource connection

1. Create a BigQuery dataset named `online_shop` that is multi-region in the United States.

Open BigQuery Studio.

Screens captured

Go to dataset created.


2. Create a Cloud Resource connection named `user_data_connection` (multi-region in the United States) and use it to create a BigLake table named `user_online_sessions` in the `online_shop` dataset.

Screens Captured

Go to connection

Copy the Service account id

Go to IAM

Grant Access

Go back to BigQuery Studio

### Task 2. Apply and verify policy tags on columns containing sensitive data

1. Use the precreated taxonomy named `access_control-taxonomy-afxl6` to apply column-level policy tags on the table.

Screens captured.

2. Verify the column-level security by running a query that omits the protected columns

SELECT * EXCEPT(zip, latitude, ip_address, longitude) FROM `qwiklabs-gcp-04-246c58bc3b91.online_shop.user_online_sessions`

### Task 3. Remove IAM permissions to Cloud Storage for other users

Follow Google best practices after migrating data to BigLake by removing IAM permissions for user 2 (`student-03-ac756809f008@qwiklabs.net`) to Cloud Storage.
Leave the IAM role for project viewer.
Remove only the IAM role for Cloud Storage.

Go to IAM

Screens captured.

Congratulations!

You have completed the Challenge Lab.
