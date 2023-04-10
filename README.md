# Overview

Portworx-Enterprise is the most widely-used and reliable cloud-native storage
solution for production workloads and provides high-availability, data
protection and security for containerized applications.
Learn more about the [Portworx Enterprise the data platform for
Kubernetes](https://portworx.com/products/portworx-enterprise/)

To learn more about the platform features, please visit our
[product features page](https://portworx.com/products/portworx-enterprise/features/)

# Pre-requisites

## Minimum GKE Node requirements:
Image type: Ubuntu

Machine type: n1-standard-4 (4 vCPUs and 4GB memory)

## Permissions:

### Cluster Access Scopes
Portworx requires permissions to create GCE PDs using the compute APIs. Also, the
GCP marketplace requires that the clusters have Read permissions for storage
APIs. These permissions can be added to the node pools from the UI when creating
the GKE cluster. If using `gcloud` you can use the following command to create a
cluster with the correct permissions:

```
gcloud container clusters create portworx-gke \
    --zone us-east1-b \
    --disk-type=pd-ssd \
    --disk-size=50GB \
    --machine-type=n1-standard-4 \
    --num-nodes=3 \
    --image-type ubuntu \
    --scopes compute-rw,storage-ro 
```

### Service Account
The service account associated with your GKE cluster should have permissions to create
and mount GCE PDs.

# Install from the Marketplace
When installing Portworx Enterprise from the GCP Marketplace it will automatically
install on all the worker nodes in your GKE cluster. Before install you can select
the type and size of GCE PDs that you want to provision and attach to each node in
the cluster. These disks will automatically get attached to the nodes on reboots
and node failures.

The marketplace installer will also create some Service Accounts to be used by
the various components. It is recommended to let the installer create these
service accounts instead of choosing pre-existing ones.

When installed from the marketplace the billing agent will automatically report
billing information based on the number of nodes to GCP. Please refer to the
marketplace listing for the pricing information.

# Install using CLI
You can also choose to install Portworx using the CLI. You will still need to
generate a license key from the GCP portal and create a Kubernetes Secret that
can be used to report billing information.

## Create Reporting Secret
Navigate to the Portworx listing on the GCP Marketplace and click on
`Configure`. Then click on the `Install via command line` tab. Here you can
generate a license key to be used for the reporting.
Choose a service account you want to be associated with the billing and
click `Generate license key`. This will download the license key to your system.
Apply the license key to your GKE cluster using the following command:
```
NS=<namespace_where_you_installed_portworx>
kubectl apply -n $NS license.yaml
```

## Install Portworx
Instructions to install Portworx using the command line can be found
[here](https://docs.portworx.com/install-portworx/kubernetes/gcp/gke-operator/)

Once Portworx has been installed please edit the Storagecluster
```
kubectl edit storagecluster -n $NS <cluster-name>
```

And add the following environment variables to the Storagecluster so that
Portworx pods can read the license generate above
```
REPORTING_SECRET: # Name of the secret generated above
REPORTING_SECRET_NAMESPACE: # Namespace where you installed Portworx

PRODUCT_PLAN_ID: # License type to be activated.
```
The last environment variable is used to select your desired Plan ID and its features and is crucial. The different Plan IDs available are:-

```PX-ENTERPRISE``` : Basic flavor of Portworx Enterprise License for VM only nodes

```PX-ENTERPRISE-DR``` : Portworx Enterprise with Disaster Recovery feature for VM only nodes

```PX-ENTERPRISE-BAREMETAL```: Px-Enterprise for both VM and Bare metal hosts

```PX-ENTERPRISE-DR-BAREMETAL```: PX-ENTERPRISE with Disaster Recovery feature for both VM and Bare metal hosts

Note:- By default, a user get a trail license with 31 days validity. 
# Upgrade

When an upgrade is available in the marketplace you can install the new version with a new name in the same namespace as the
previous install. This will replace all the components for the Application with the updated version.

You can remove the old version of the Application once all the components in the new version are Running.

# Uninstall
If you’re using the Portworx Operator, you can uninstall Portworx by adding a delete strategy to your StorageCluster object, then deleting it. When uninstalling, you may choose to either keep the the data on your drives, or wipe them completely:

- Uninstall: will remove the Kubernetes objects, Portworx systemctl service, /etc/pwx and /opt/pwx directories, and all traces of Portworx on the nodes. The drives will not be formatted and none the Portworx Metadata in the KVDB will be deleted. You may need to Uninstall Portworx if you installed Portworx in the wrong namespace.
- Uninstall and wipe: will remove all of the resources listed in the “Uninstall” procedure, and also removes (formats) all data from your disks permanently, including the Portworx metadata. You may want to perform an uninstall and wipe when you decommission a cluster.

1. Enter the kubectl edit command to modify your storage cluster and specify your namespace:
```
  kubectl edit -n kube-system storagecluster <storagecluster-name>
```
2. Modify your StorageCluster object, adding the deleteStrategy field with either the Uninstall or UninstallAndWipe type:
```
spec:       
  deleteStrategy:
    type: # Uninstall or UninstallAndWipe
```

3. Enter the kubectl delete command, specifying the name of your StorageCluster object and specify your namespace:

```
kubectl delete StorageCluster <your-storagecluster-name> -n kube-system
```
NOTE: This operation can take several minutes to complete.

4. Delete the operator:
```
kubectl delete deployment -n <namespace> portworx-operator
```