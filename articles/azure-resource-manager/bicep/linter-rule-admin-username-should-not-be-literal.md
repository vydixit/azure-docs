---
title: Linter rule - admin user name should not be literal
description: Linter rule - admin user name should not be a literal
ms.topic: conceptual
ms.date: 10/15/2021
---

# Linter rule - admin user name should not be literal

This rule finds when an admin user name is set to a literal value.

## Returned code

`adminusername-should-not-be-literal`

## Solution

Don't use a literal value or an expression that evaluates to a literal value. Instead, create a parameter for the user name and assign it to the admin user name.

The following example fails this test because the user name is a literal value.

```bicep
resource vm 'Microsoft.Compute/virtualMachines@2020-12-01' = {
  name: 'name'
  location: location
  properties: {
    osProfile: {
      adminUsername: 'adminUsername'
    }
  }
}
```

The next example fails this test because the expression evaluates to a literal value when the default value is used.

```bicep
var defaultAdmin = 'administrator'
resource vm 'Microsoft.Compute/virtualMachines@2020-12-01' = {
  name: 'name'
  location: location
  properties: {
    osProfile: {
      adminUsername: defaultAdmin
    }
  }
}
```

This example passes this test.

```bicep
@secure()
param adminUsername string
param location string
resource vm 'Microsoft.Compute/virtualMachines@2020-12-01' = {
  name: 'name'
  location: location
  properties: {
    osProfile: {
      adminUsername: adminUsername
    }
  }
}
```

## Next steps

For more information about the linter, see [Use Bicep linter](./linter.md).
