# CloudBolt Collector Helm Chart

The `cloudbolt-collector` chart deploys an application that extracts Kubernetes cluster utilization data 
and sends it to the CloudBolt platform.

## Requirements

- **Helm 3:** To install Helm, run the following commands:
  ```console
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh
  ```

## Configuration Parameters

Below is a table of various parameters that can be set in the `values.yaml` file of the `cloudbolt-collector-helm` chart, along with their required status and default values:

| Helm Parameter                     | Required   | Description                          | Default Value       |
|------------------------------------|------------|--------------------------------------|---------------------|
| `INGESTION_API_URL`                | Yes        | Ingestion API URL                    | `""` (empty string) |
| `DEBUG`                            | No         | Debug mode flag                      | `""` (empty string) |
| `IMAGE_VERSION`                    | No         | Version of the image                 | `"<helm-chart-version>"`|
| `PROMETHEUS_BASE_URL`              | No         | URL to prometheus service            | `""` (defaults to OpenShift prometheus URL) |
| `COREAPI_BASE_URL`.                | No         | URL to core api.                     | `""` (defaults to OpenShift coreapi URL) |

## Prerequisites

Before you begin, ensure you have access to an OpenShift Cluster and have Helm installed in your environment.

## Installation Steps

### 1. Login to Your OpenShift Cluster

Execute the following command to log in to your OpenShift Cluster. Replace `<username>`, `<cluster-ip>`, and `<cluster-port>` with your actual OpenShift credentials and cluster information.

```console
oc login --username <username> <cluster-ip>:<cluster-port>
```

### 2. Add the CloudBolt Helm Repository

Add the CloudBolt Collector Helm repository to your Helm configuration:

```console
helm repo add cloudbolt-collector https://cloudboltsoftware.github.io/cloudbolt-collector-helm/
helm repo update
```

### 3. Create a New Namespace/Project

Create a new project or namespace in your OpenShift cluster:

```console
oc new-project cloudbolt-collector
```

### 4. Select the project

Select the project in which we are going deploy with following command:

```console
oc project cloudbolt-collector
```

### 5. Set Environmental Variables

Replace `<placeholders>` with appropriate values:

```console
export IMAGE_VERSION="<release-version>"  # Default is chart version
export INGESTION_API_URL="<ingestion-api-url>"
```

### 6. Create Secrets for API Access

Create the necessary secrets for API access:

```console
oc create secret generic cb-ingestion-token \
  --from-literal=INGESTION_API_TOKEN=<ingestion-api-token> \
  -n cloudbolt-collector
```

### 7. Install the Chart

Install the latest CloudBolt Collector Helm chart from the `cloudbolt-collector` repository. 
If you want to install version `v0.20.0`, you can specify it using the `--version` flag:

```console
helm install cloudbolt-collector cloudbolt-collector/cloudbolt-collector \
  --set INGESTION_API_URL=$INGESTION_API_URL
```

### Upgrade the Chart

To upgrade the CloudBolt Collector Helm chart to a newer version, first ensure your local Helm repository index is updated:

```console
helm repo update cloudbolt-collector
```

Then, upgrade the release to the desired version. If you want to upgrade to the latest version available in the repository, use the following command:

```console
helm upgrade cloudbolt-collector cloudbolt-collector/cloudbolt-collector \
  --set INGESTION_API_URL=$INGESTION_API_URL
```

If you wish to upgrade to a specific version, use the `--version` flag:

```console
helm upgrade cloudbolt-collector cloudbolt-collector/cloudbolt-collector \
  --version v0.21.0 \
  --set IMAGE_VERSION=$IMAGE_VERSION \
  --set INGESTION_API_URL=$INGESTION_API_URL
```

## Installation on Microsoft Azure

This helm chart was also succesfully used on Azure's Kubernetes cluster (AKS) and might even work on other Kubernetes offerings.

As an additional prerequisite a Prometheus installation scraping node and pod metrics is required.

```console
export PROMETHEUS_BASE_URL="http://prometheus-kube-prometheus-prometheus.monitoring.svc.cluster.local:9090"
export COREAPI_BASE_URL="https://kubernetes.default.svc"
export INGESTION_API_URL="https://gn36i03mq1.execute-api.eu-west-2.amazonaws.com/v1/data/ingest"

# create a secret with the CloudBolt API token
kubectl create secret generic cb-ingestion-token \
  --from-literal=INGESTION_API_TOKEN=<cloudbolt-ingestion-api-token> \
  -n cloudbolt-collector

# Updates or installs the cloudbolt-collector helm chart
helm upgrade --install cloudbolt-collector ./ -f values.yaml \
  --namespace cloudbolt-collector --create-namespace \
  --set INGESTION_API_URL=$INGESTION_API_URL \
  --set PROMETHEUS_BASE_URL=$PROMETHEUS_BASE_URL \
  --set COREAPI_BASE_URL=$COREAPI_BASE_URL \
  --set clusterRole.create=true \
  --set securityContext.runAsUser=1001
```
