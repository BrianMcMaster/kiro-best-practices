---
title: GCP CLI Best Practices
inclusion: always
---

# GCP CLI Best Practices

## Output Formatting
- Use `--format=json` for programmatic processing and scripts
- Use `--format=table` for human-readable console output
- Use `--format=csv` for comma-separated values in scripts
- Use `--filter` and `--sort-by` to refine results

Example:
```bash
gcloud compute instances list --format=table
gcloud compute instances list --format=json --filter="status:RUNNING"
gcloud projects list --format="csv(projectId,name)" --sort-by=name
```

## Authentication and Security
- Use `gcloud auth login` for interactive authentication
- Use service account keys for automation (store securely)
- Use `gcloud auth application-default login` for application development
- Set project context explicitly with `gcloud config set project`

```bash
# Interactive login
gcloud auth login

# Service account authentication
gcloud auth activate-service-account --key-file=path/to/keyfile.json

# Set default project
gcloud config set project my-project-id

# List available projects
gcloud projects list --format="table(projectId,name)"
```

## Configuration Management
- Use `gcloud config configurations` to manage multiple environments
- Set default regions and zones to avoid repetitive flags
- Use `gcloud config list` to verify current settings
- Store configuration in named configurations for different contexts

```bash
# Create and activate configuration
gcloud config configurations create dev-environment
gcloud config configurations activate dev-environment

# Set defaults for current configuration
gcloud config set compute/region us-central1
gcloud config set compute/zone us-central1-a

# List all configurations
gcloud config configurations list
```

## Resource Management
- Use consistent naming conventions and labels
- Apply labels for cost tracking and resource organization
- Use `--dry-run` flag when available for testing
- Specify full resource paths to avoid ambiguity

```bash
# Create instance with labels
gcloud compute instances create my-instance \
  --zone=us-central1-a \
  --labels=environment=dev,team=backend

# List resources with filtering
gcloud compute instances list --filter="labels.environment=dev"
```

## Error Handling and Debugging
- Use `--verbosity=debug` for detailed troubleshooting
- Use `--log-http` to see HTTP requests and responses
- Check command exit codes in scripts
- Use `--async` for long-running operations when appropriate

```bash
# Debug mode for troubleshooting
gcloud compute instances create my-instance --verbosity=debug

# Log HTTP requests
gcloud compute instances list --log-http

# Async operation
gcloud compute instances create my-instance --async
```

## Performance and Efficiency
- Use `--async` flag for long-running operations
- Batch operations using `gcloud compute instances add-metadata` for multiple instances
- Use `--limit` to control result set size
- Cache authentication for repeated operations

```bash
# Async deployment
gcloud deployment-manager deployments create my-deployment --config=config.yaml --async

# Limit results
gcloud compute instances list --limit=10

# Check operation status
gcloud compute operations describe operation-name --zone=us-central1-a
```

## Scripting and Automation
- Use `--quiet` flag to suppress interactive prompts
- Set `CLOUDSDK_CORE_DISABLE_PROMPTS=1` environment variable
- Use `--format="value(field)"` to extract specific values
- Handle pagination with `--page-size` and `--limit`

```bash
# Suppress prompts in scripts
export CLOUDSDK_CORE_DISABLE_PROMPTS=1
gcloud compute instances delete my-instance --quiet

# Extract specific values
INSTANCE_IP=$(gcloud compute instances describe my-instance \
  --zone=us-central1-a --format="value(networkInterfaces[0].accessConfigs[0].natIP)")
```