# Azure CLI Cheatsheet

## Contents

## General

Some `az` parameters can be used all the time, add them as you need to the commands listed below.

|Parameter|Description|
|------|------------|
| `--output table` | format output as human readable (default is json) |
| `--output tsv` | format output as tab seperated values |

## Subscriptions / Accounts

[How to manage Azure subscriptions with the Azure CLI](https://docs.microsoft.com/en-us/cli/azure/manage-azure-subscriptions-azure-cli)

|Command|Description|
|------|------------|
| `az account show` | show data of the current |
| `az account list` | list all available subscriptions |
| `az account set <subscription-id>` | set the default subscription of the CLI |

## Tutorials

- [Build and run a custom image in Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/tutorial-custom-container?pivots=container-linux)