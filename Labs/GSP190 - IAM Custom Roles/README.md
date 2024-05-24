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

Create your role definition YAML file `role-definition.yaml`:

```js role-definition.yaml
title: "Role Editor"
description: "Edit access for App Versions"
stage: "ALPHA"
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
```

Execute the following `gcloud` command to create the custom role:

```
gcloud iam roles create editor --project $DEVSHELL_PROJECT_ID \
--file role-definition.yaml
```
Execute the following `gcloud` command to create a new role using flags:

```
gcloud iam roles create viewer --project $DEVSHELL_PROJECT_ID \
--title "Role Viewer" --description "Custom role description." \
--permissions compute.instances.get,compute.instances.list --stage ALPHA
```

### Task 5. List the custom roles

Execute the following `gcloud` command to list custom roles, specifying either project-level or organization-level custom roles:

```
gcloud iam roles list --project $DEVSHELL_PROJECT_ID
```

Execute the following `gcloud` command to list predefined roles:

```
gcloud iam roles list
```

### Task 6. Update an existing custom role

#### Update a custom role using a YAML file

Get the current definition for the role by executing the following `gcloud` command:

```
gcloud iam roles describe editor --project $DEVSHELL_PROJECT_ID
```

Create a `new-role-definition.yaml` file with the output from last command.

```js new-role-definition.yaml
description: Edit access for App Versions
etag: BwYZLgnldeQ=
includedPermissions:
- appengine.versions.create
- appengine.versions.delete
name: projects/qwiklabs-gcp-02-a01c0a24da96/roles/editor
stage: ALPHA
title: Role Editor
```

Add the following permissions under `includedPermissions`:

```
- storage.buckets.get
- storage.buckets.list
```

Execute the following `gcloud` command to update the role:

```
gcloud iam roles update editor --project $DEVSHELL_PROJECT_ID \
--file new-role-definition.yaml
```

#### Update a custom role using flags

Execute the following `gcloud` command to add permissions to the viewer role using flags:

```
gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--add-permissions storage.buckets.get,storage.buckets.list
```

### Task 7. Disable a custom role

Execute the following `gcloud` command to disable the viewer role:

```
gcloud iam roles update viewer --project $DEVSHELL_PROJECT_ID \
--stage DISABLED
```

### Task 8. Delete a custom role

Use the `gcloud iam roles delete` command to delete a custom role. Once deleted the role is inactive and cannot be used to create new IAM policy bindings:

```
gcloud iam roles delete viewer --project $DEVSHELL_PROJECT_ID
```

### Task 9. Restore a custom role

Within the 7 days window you can restore a role. Deleted roles are in a **DISABLED** state. To make it available again, update the `--stage` flag:

```
gcloud iam roles undelete viewer --project $DEVSHELL_PROJECT_ID
```

### Congratulations!

You have completed the lab.
