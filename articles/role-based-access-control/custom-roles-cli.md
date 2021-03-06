---
title: Create custom roles using Azure CLI | Microsoft Docs
description: Learn how to create custom roles for role-based access control (RBAC) using Azure CLI. This includes how to list, create, update, and delete custom roles.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman

ms.assetid: 3483ee01-8177-49e7-b337-4d5cb14f5e32
ms.service: role-based-access-control
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 06/20/2018
ms.author: rolyon
ms.reviewer: bagovind
---
# Create custom roles using Azure CLI

If the [built-in roles](built-in-roles.md) don't meet the specific needs of your organization, you can create your own custom roles. This article describes how to create and manage custom roles using Azure CLI.

## Prerequisites

To create custom roles, you need:

- Permissions to create custom roles, such as [Owner](built-in-roles.md#owner) or [User Access Administrator](built-in-roles.md#user-access-administrator)
- [Azure CLI](/cli/azure/install-azure-cli) installed locally

## List custom roles

To list custom roles that are available for assignment, use [az role definition list](/cli/azure/role/definition#az-role-definition-list). The following examples list all the custom roles in the current subscription.

```azurecli
az role definition list --custom-role-only true --output json | jq '.[] | {"roleName":.roleName, "roleType":.roleType}'
```

```azurecli
az role definition list --output json | jq '.[] | if .roleType == "CustomRole" then {"roleName":.roleName, "roleType":.roleType} else empty end'
```

```Output
{
  "roleName": "My Management Contributor",
  "type": "CustomRole"
}
{
  "roleName": "My Service Reader Role",
  "type": "CustomRole"
}
{
  "roleName": "Virtual Machine Operator",
  "type": "CustomRole"
}

...
```

## Create a custom role

To create a custom role, use [az role definition create](/cli/azure/role/definition#az-role-definition-create). The role definition can be a JSON description or a path to a file containing a JSON description.

```azurecli
az role definition create --role-definition <role_definition>
```

The following example creates a custom role named *Virtual Machine Operator*. This custom role assigns access to all read operations of *Microsoft.Compute*, *Microsoft.Storage*, and *Microsoft.Network* resource providers and assigns access to start, restart, and monitor virtual machines. This custom role can be used in two subscriptions. This example uses a JSON file as an input.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111",
    "/subscriptions/33333333-3333-3333-3333-333333333333"
  ]
}
```

```azurecli
az role definition create --role-definition ~/roles/vmoperator.json
```

## Update a custom role

To update a custom role, first use [az role definition list](/cli/azure/role/definition#az-role-definition-list) to retrieve the role definition. Second, make the desired changes to the role definition. Finally, use [az role definition update](/cli/azure/role/definition#az-role-definition-update) to save the updated role definition.

```azurecli
az role definition update --role-definition <role_definition>
```

The following example adds the *Microsoft.Insights/diagnosticSettings/* operation to the *Actions* of the *Virtual Machine Operator* custom role.

vmoperator.json

```json
{
  "Name": "Virtual Machine Operator",
  "IsCustom": true,
  "Description": "Can monitor and restart virtual machines.",
  "Actions": [
    "Microsoft.Storage/*/read",
    "Microsoft.Network/*/read",
    "Microsoft.Compute/*/read",
    "Microsoft.Compute/virtualMachines/start/action",
    "Microsoft.Compute/virtualMachines/restart/action",
    "Microsoft.Authorization/*/read",
    "Microsoft.Resources/subscriptions/resourceGroups/read",
    "Microsoft.Insights/alertRules/*",
    "Microsoft.Insights/diagnosticSettings/*",
    "Microsoft.Support/*"
  ],
  "NotActions": [

  ],
  "AssignableScopes": [
    "/subscriptions/11111111-1111-1111-1111-111111111111",
    "/subscriptions/33333333-3333-3333-3333-333333333333"
  ]
}
```

```azurecli
az role definition update --role-definition ~/roles/vmoperator.json
```

## Delete a custom role

To delete a custom role, use [az role definition delete](/cli/azure/role/definition#az-role-definition-delete). To specify the role to delete, use the role name or the role ID. To determine the role ID, use [az role definition list](/cli/azure/role/definition#az-role-definition-list).

```azurecli
az role definition delete --name <role_name or role_id>
```

The following example deletes the *Virtual Machine Operator* custom role.

```azurecli
az role definition delete --name "Virtual Machine Operator"
```

## Next steps

- [Tutorial: Create a custom role using Azure CLI](tutorial-custom-role-cli.md)
- [Custom roles in Azure](custom-roles.md)
- [Azure Resource Manager resource provider operations](resource-provider-operations.md)