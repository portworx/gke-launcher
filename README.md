# Overview

Portworx-Enterprise is the most widely-used and reliable cloud-native storage
solution for production workloads and provides high-availability, data
protection and security for containerized applications.
Learn more about the [Portworx Enterprise the data platform for
Kubernetes](https://portworx.com/products/introduction/)

To learn more about the platform features, please visit our
[product features page](https://portworx.com/products/features/)

# Pre-requisites

## Minimum GKE Node requirements:
Image type: Ubuntu

Machine type: n1-standard-4 (4 vCPUs and 4GB memory)

## Permissions:

### Cluster Access Scopes
Portworx requires permissions to create GCE PDs using the compute APIs. Also, the
GCP market place requires that the clusters have Read permissions for storage
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
the various componenets. It is recommended to let the installer create these
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
[here](https://docs.portworx.com/portworx-install-with-kubernetes/cloud/gke)

Once Portworx has been installed please edit the `portworx` Daemonset 
```
kubectl edit daemonset -n $NS portworx
```

And add the following environment variables to the `portworx` container so that
Portworx can read the license generate above
```
REPORTING_SECRET: Name of the secret generated above
REPORTING_SECRET_NAMESPACE: Namespace where you installed Portworx
```

# Upgrade

When an upgrade is available in the marketplace you can install the new version with a new name in the same namespace as the
previous install. This will replace all the components for the Application with the updated version.

You can remove the old version of the Application once all the components in the new version are Running.

# Uninstall

**WARNING: Uninstalling Portworx is a destructive process and you will not be able to recover volumes provisioned by
Portworx. Please use this with caution.**

You can uninstall Portworx from your GKE cluster by running the following command:
```
curl -fsL https://install.portworx.com/px-wipe | bash
```

This will remove all Portworx specific state from the nodes and clean all the disks used by Portworx.
Once the above script completes succesfully you can delete the Application object created by GCP.

You can find more information [here].(https://docs.portworx.com/portworx-install-with-kubernetes/operate-and-maintain-on-kubernetes/uninstall/uninstall/#uninstall)
