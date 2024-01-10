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

| Parameter                          | Required   | Description                          | Default Value       |
|------------------------------------|------------|--------------------------------------|---------------------|
| `DEBUG`                            | No         | Debug mode flag                      | `""` (empty string) |
| `IMAGE_VERSION`                    | No         | Version of the image                 | `"<helm-chart-version>"`|
| `OCP_IP`                           | Yes        | OpenShift Cluster IP                 | `""` (empty string) |
| `OCP_PORT`                         | Yes        | OpenShift Cluster Port               | `""` (empty string) |
| `OCP_SERVICENAME`                  | Yes        | OpenShift Service Name               | `""` (empty string) |
| `OCP_ENABLE_SSL_VERIFICATION`      | No         | SSL Verification for HTTP and HTTPS  | `""` (empty string) |
| `INGESTION_API_URL`                | Yes        | Ingestion API URL                    | `""` (empty string) |

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

### 4. Create a New ServiceAccount and assign a new role

Create a new serviceaccount and assign it a new role in your OpenShift cluster:

```console
oc create sa <service-account-name>
```

```console
oc adm policy add-cluster-role-to-user cluster-monitoring-view -z <service-account-name> -n cloudbolt-collector
```

### 5. Set Namespace/Project Annotations

Annotate the newly created namespace:

```console
oc annotate --overwrite namespace cloudbolt-collector openshift.io/sa.scc.uid-range='1000/1000' openshift.io/sa.scc.supplemental-groups='1000/1000'
```

### 6. Set Environmental Variables

Replace `<placeholders>` with appropriate values:

```console
export IMAGE_VERSION="<release-version>"  # Default is chart version
export OCP_IP="<openshift-cluster-ip>"
export OCP_PORT="<openshift-cluster-port>"
export OCP_SERVICENAME="<ocp-username>"
export OCP_ENABLE_SSL_VERIFICATION="<SSL Verification for http and https>"
export INGESTION_API_URL="<ingestion-api-url>"
export SERVICE_ACCOUNT_NAME="<service-account-name>"
```

### 7. Create Secrets for API Access

Create the necessary secrets for API access:

```console
oc create secret generic my-ocp-secret \
  --from-literal=OCP_SERVICEPASS=<ocp-password> \
  -n cloudbolt-collector

oc create secret generic cb-ingestion-token \
  --from-literal=INGESTION_API_TOKEN=<ingestion-api-token> \
  -n cloudbolt-collector
```
### 8. Install the Chart

Install the latest CloudBolt Collector Helm chart from the `cloudbolt-collector` repository. 
If you want to install version `v0.20.0`, you can specify it using the `--version` flag:

```console
helm install cloudbolt-collector cloudbolt-collector/cloudbolt-collector \
  --set OCP_IP=$OCP_IP \
  --set OCP_PORT=$OCP_PORT \
  --set OCP_SERVICENAME=$OCP_SERVICENAME \
  --set OCP_ENABLE_SSL_VERIFICATION=$OCP_ENABLE_SSL_VERIFICATION \
  --set INGESTION_API_URL=$INGESTION_API_URL \
  --set SERVICE_ACCOUNT_NAME=$SERVICE_ACCOUNT_NAME
```

### Upgrade the Chart

To upgrade the CloudBolt Collector Helm chart to a newer version, first ensure your local Helm repository index is updated:

```console
helm repo update cloudbolt-collector
```

Then, upgrade the release to the desired version. If you want to upgrade to the latest version available in the repository, use the following command:

```console
helm upgrade cloudbolt-collector cloudbolt-collector/cloudbolt-collector \
  --set OCP_IP=$OCP_IP \
  --set OCP_PORT=$OCP_PORT \
  --set OCP_SERVICENAME=$OCP_SERVICENAME \
  --set OCP_ENABLE_SSL_VERIFICATION=$OCP_ENABLE_SSL_VERIFICATION \
  --set INGESTION_API_URL=$INGESTION_API_URL \
  --ser SERVICE_ACCOUNT_NAME=$SERVICE_ACCOUNT_NAME
```

If you wish to upgrade to a specific version, use the `--version` flag:

```console
helm upgrade cloudbolt-collector cloudbolt-collector/cloudbolt-collector \
  --version v0.21.0 \
  --set IMAGE_VERSION=$IMAGE_VERSION \
  --set OCP_IP=$OCP_IP \
  --set OCP_PORT=$OCP_PORT \
  --set OCP_SERVICENAME=$OCP_SERVICENAME \
  --set OCP_ENABLE_SSL_VERIFICATION=$OCP_ENABLE_SSL_VERIFICATION \
  --set INGESTION_API_URL=$INGESTION_API_URL \
  --set SERVICE_ACCOUNT_NAME=$SERVICE_ACCOUNT_NAME
```
