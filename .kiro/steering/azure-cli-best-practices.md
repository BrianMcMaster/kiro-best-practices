---
title: Azure CLI Best Practices
inclusion: always
---

# Azure CLI Best Practices

## Output Formatting
- Use `--output json` for programmatic processing and scripts
- Use `--output table` for human-readable console output
- Use `--output tsv` for tab-separated values in scripts
- Use `--query` with JMESPath to filter and transform results

Example:
```bash
az vm list --output table
az vm list --output json --query "[].{Name:name, Status:powerState}"
az resource list --output tsv --query "[].{Name:name, Type:type}"
```

## Authentication and Security
- Use `az login` with service principals for automation
- Prefer managed identities over service principal credentials
- Use `az account set` to switch between subscriptions
- Store credentials securely, never in code or scripts

```bash
# Service principal login for automation
az login --service-principal -u $CLIENT_ID -p $CLIENT_SECRET --tenant $TENANT_ID

# List and set subscription
az account list --output table
az account set --subscription "subscription-name-or-id"
```

## Resource Management
- Use resource groups to organize related resources
- Apply consistent naming conventions and tags
- Use `--dry-run` parameter when available for testing
- Always specify resource group and location explicitly

```bash
# Create resource group with tags
az group create --name myResourceGroup --location eastus --tags Environment=Dev Project=MyApp

# List resources with filtering
az resource list --resource-group myResourceGroup --output table
```

## Error Handling and Debugging
- Use `--debug` flag for detailed troubleshooting information
- Check command exit codes in scripts
- Use `--verbose` for additional operational details
- Handle rate limiting and retry logic in automation

```bash
# Debug mode for troubleshooting
az vm create --debug --name myVM --resource-group myRG --image UbuntuLTS

# Verbose output
az deployment group create --verbose --resource-group myRG --template-file template.json
```

## Performance and Efficiency
- Use `--no-wait` for long-running operations when appropriate
- Batch operations when possible to reduce API calls
- Use parallel execution for independent operations
- Cache authentication tokens for repeated operations

```bash
# Non-blocking deployment
az vm create --no-wait --name myVM --resource-group myRG --image UbuntuLTS

# Check operation status
az vm wait --created --name myVM --resource-group myRG
```

## Configuration Management
- Use `az configure` to set default values
- Store frequently used parameters in variables
- Use configuration files for complex deployments
- Keep CLI version updated for latest features

```bash
# Set default resource group and location
az configure --defaults group=myResourceGroup location=eastus

# Use environment variables
export RESOURCE_GROUP="myResourceGroup"
az vm list --resource-group $RESOURCE_GROUP
```