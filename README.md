# Provisioning an Azure DevOps organization using Terraform

This repository contains an example about how you could provision an Azure DevOps Team organization using the official terraform provider   

## Prerequisites
There are some prerequisites that we need to fulfill in order to ran this example.
- Having an existing AzDo organization.
- Having your AzDo organization connected to an AAD directory.
  - That's needed because we're not going to create new users, we're going to use AAD groups and enrolls the users on those groups into our AzDo organization.


## Terraform folder structure

The repository is divided by:
- **/project** folder : Creates the team projects for every team.
- **/entitlements** folder: Enrolls the AAD users from a given AAD group and assign them a license.
  - It uses the module **add-entitlement-to-group-users** found in the **/modules** folder.
- **/groups-permissions** folder: Reads the given groups from AAD and adds them into the given security group of a given team project.
  - It uses the module **add-aad-groups-to-azdo-team-project-sec-group** found in the **/modules** folder.
- **/repository** folder: Creates the git repositories for the apps and set a few master branch policies
  - It uses the module **create-repository-and-branch-policies** found in the **/modules** folder.


## Terraform modules 


### _add-entitlement-to-group-users_ module

### Parameters:

| **Variables**    | **Type**     | **Description**                                                             |
|------------------|--------------|-----------------------------------------------------------------------------|
| aad_users_groups | list(string) | List of AAD groups that are going to be enrolled into the AzDo organization.|
| license_type     | string       | Which type of license is going to be assigned to the users.                 |

### Usage:

Given:
- An AAD group
- A license type

It enrolls all the members of the AAD group into the AzDo organization and assigns them a license from the specified type

### Example: 

```yaml
module "add-entitlement-to-sales-team-group-users" {
    source      = "../modules/add-entitlement-to-group-users"
    aad_users_groups = ["it-sales-team"]
    license_type = "basic"
}
```

### _add-aad-groups-to-azdo-team-project-sec-group_ module

### Parameters:

| **Variables**    | **Type**     | **Description**                                                             |
|------------------|--------------|-----------------------------------------------------------------------------|
| project_name     | string       | Name of the AzDo Team project where the group is going to be added.         |
| azdo_group_name  | string       | Which team project permissions group to use.  Ex: Contributors, Readers,... |
| aad_users_groups | list(string) | List of AAD groups that are going to be added into the "azdo_group_name"    |

### Usage:

Given:
- A Team Project
- An AzDo Team Project permissions group (Readers, Contributors)
- An AAD group

It retrieves the group from the AAD an adds them into a specific team project permissions group.
Right now it only works with the default project permissions groups, you cannot assign them into a custom permissions group.

### Example: 

```yaml
module "add-users-for-sales-team-to-azdo-group" {
    source      = "../modules/add-aad-users-to-azdo-group"
    project_name =  "Commercial Team Project"
    azdo_group_name = "Contributors"
    aad_users_groups = ["it-commercial-team"]
}
```

### _create-repository-and-branch-policies_ module

### Parameters:

| **Variables**    | **Type**     | **Description**                                                             |
|------------------|--------------|-----------------------------------------------------------------------------|
| project_name     | string       | Name of the AzDo Team project where the repository is going to be created.  |
| repository_name  | string       | Name of the repository.                                                     |

### Usage:

Given:
- A Team project
- A Git Repository

It creates the Git Repository in the given Team Project.   
It also creates the master branch policies. Those branch policies cannot be configured via module variables for now.

### Example: 

```yaml
module "create-repository-and-policies-for-commercial-team-api" {
    source      = "../modules/create-repository-and-branch-policies"
    project_name = "Commercial Team Project"
    repository_name = "comm-web-api"
}
```

