# cloudbolt-collector-helm

Deploys a Kubernetes pod to gather data and create reports on resource usage, expenses, and other potential aspects.

## Requirements
- [Helm Chart](https://helm.sh/docs/)
To Install Helm Chart run these commands in your environment:
```console
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
```
```console
chmod 700 get_helm.sh
```
```console
./get_helm.sh
```


## How to install this chart
Login to your OpenShift Cluster:

```console
oc login --username <username> <cluster-ip>:<cluster-port>
```
Clone the repo:

```console
git clone git@github.com:CloudBoltSoftware/cloudbolt-collector-helm.git
```

Change directory into the project (only if you have taken the GitHub pull):

```console
cd cloudbolt-collector-helm
```
Set the environmental variables required for Helm Chart deployment (If not present in the environment) :

```console
export OCP_IP="<openshift-cluster-ip>"
export OCP_PORT="<openshift-cluster-port>"
export OCP_SERVICENAME="<username>"
export OCP_SERVICEPASS="<password>"
export OCP_ENABLE_SSL_VERIFICATION="<SSL Verification for http and https>"
export INGESTION_API_TOKEN="<ingestion-api-access-token>"
```
To set or change version of docker image (by default latest):
```console
export IMAGE_VERSION="<version>"
```

To install the chart with the `env-values`:

```console
helm install cloudbolt-collector-helm . --set IMAGE_VERSION=$IMAGE_VERSION,OCP_IP=$OCP_IP,OCP_PORT=$OCP_PORT,
OCP_SERVICENAME=$OCP_SERVICENAME,OCP_SERVICEPASS=$OCP_SERVICEPASS,OCP_ENABLE_SSL_VERIFICATION=$OCP_ENABLE_SSL_VERIFICATION,INGESTION_API_TOKEN=$INGESTION_API_TOKEN
```
