# [GSP190 - IAM Custom Roles](https://www.cloudskillsboost.google/games/5058/labs/33036)

## Overview
Cloud IAM provides the right tools to manage resource permissions with minimum fuss and high automation. You don't directly grant users permissions. Instead, you grant them roles, which bundle one or more permissions. This allows you to map job functions within your company to groups and roles. Users get access only to what they need to get the job done, and admins can easily grant default permissions to entire groups of users.

There are two kinds of roles in Cloud IAM:
- Predefined Roles*
- Custom Roles

**Predefined roles** are created and maintained by Google. Their permissions are automatically updated as necessary, such as when new features or services are added to Google Cloud.

**Custom roles** are user-defined, and allow you to bundle one or more supported permissions to meet your specific needs. Custom roles are not maintained by Google; when new permissions, features, or services are added to Google Cloud, your custom roles will not be updated automatically.You create a custom role by combining one or more of the available Cloud IAM permissions. Permissions allow users to perform specific actions on Google Cloud resources.


## Solution

### Setup

```
gcloud config set compute/region us-east1
```

### Task 1. View the available permissions for a resource

Run the following to get the list of permissions available for your project.

```
gcloud iam list-testable-permissions //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
```

### Task 2. Get the role metadata

To view the role metadata, use command below, replacing [ROLE_NAME] with the role. For example: roles/viewer or roles/editor:

```
gcloud iam roles describe [ROLE_NAME]
```

### Task 3. View the grantable roles on resources

Execute the following gcloud command to list grantable roles from your project:

```
gcloud iam list-grantable-roles //cloudresourcemanager.googleapis.com/projects/$DEVSHELL_PROJECT_ID
```

### Task 4. Create a custom role

#### Create a custom role using a YAML file

Create your role definition YAML file: role-definition.yaml

```role-definition.yaml
title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
```



###


###
