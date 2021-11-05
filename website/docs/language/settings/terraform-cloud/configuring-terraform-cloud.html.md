---
layout: "language"
page_title: "Configuring Terraaform Cloud"
sidebar_current: "terraform-cloud-configuration"
description: "Configuring Terraform Cloud"
---

# Configuring Terraform Cloud

To enable the [CLI-driven run workflow](https://www.terraform.io/docs/cloud/run/cli.html), a
Terraform configuration can integrate Terraform Cloud via a special `cloud` block within the
top-level `terraform` block, e.g.:

```
terraform {
  cloud {
    organization = "my-org"
    workspaces {
      tags = ["networking"]
    }
  }
}
```

Using the Cloud integration is mutually exclusive of declaring any backend; that is, a configuration
can only declare one or the other. Similar to backends...

- A configuration can only provide one cloud block.
- A cloud block cannot refer to named values (like input variables, locals, or data source attributes).

## Configuration variables

The following configuration options are supported:

* `organization` - (Required) The name of the organization containing the
  workspace(s) the current configuration should be mapped to.
* `workspaces` - (Required) A block declaring a strategy for mapping local CLI workspaces to remote
  Terraform Cloud workspaces.
  The `workspaces` block supports the following keys, each denoting a 'strategy':

  * `tags` - (Optional) TODO

  * `name` - (Optional) The full name of one remote workspace. When configured,
    only the default workspace can be used. This option conflicts with `prefix`.

    For more details on using workspaces, this configuration block, and mapping strategies, see [Workspaces](/docs/language/settings/terraform-cloud/workspaces.html)
* `hostname` - (Optional) The hostname of a Terraform Enterprise installation, if using Terraform
  Enterprise. Defaults to Terraform Cloud (app.terraform.io).
* `token` - (Optional) The token used to authenticate with Terraform Cloud.
  We recommend omitting the token from the configuration, and instead using
  [`terraform login`](/docs/cli/commands/login.html) or manually configuring
  `credentials` in the
  [CLI config file](/docs/cli/config/config-file.html#credentials).

## Example Configurations

### Basic Configuration

```hcl
terraform {
  cloud {
    organization = "company"
    workspaces {
      tags = ["networking", "source:cli"]
    }
  }
}
```

In the example above, all Terraform Cloud workspaces with the `networking` and `source:cli` tags
will be mapped to the current configuration. `terraform workspace new example` would similarly
create a new Terraform Cloud workspace named `example` tagged with `networking` and `source:cli`.

### Configurating a single workspace

```hcl
terraform {
  cloud {
    organization = "company"
    workspaces {
      name = "networking-prod-us-east"
    }
  }
}
```

In the example above, the `workspaces` block maps the current configuration to a single specific
Terraform Cloud workspace named `networking-prod-us-east`; Terraform will create this workspace if
it does not yet exist when running `terraform init`. Note that using a particular workspace in this
way means that commands which utilize multiple workspaces have no effect (e.g. `terraform workspace
new`, `terraform workspace select`, etc).

### Using Partial Configuration

Like a state backend, the `cloud` option supports [partial
configuration](/docs/language/settings/backends/configuration.html#partial-configuration) with the
`-backend-config` flag, allowing you to supply configuration values from separate file.

```hcl
# main.tf
terraform {
  required_version = "~> 1.1.0"

  cloud {}
}
```

Configuration file:

```hcl
# config.cloud
workspaces { tags = ["networking"] }
hostname     = "app.terraform.io"
organization = "company"
```

Running `terraform init` with the configuration file:

```sh
$ terraform init -backend-config=config.cloud
```

->  **Note:** All of the configurations above omit an authentication token. We recommend omitting
the token from the configuration, and instead using [`terraform login`](/docs/cli/commands/login.html)
or manually configuring `credentials` in the [CLI config file](/docs/cli/config/config-file.html#credentials).

