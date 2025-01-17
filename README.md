# qiot-manufacturing-factory-installer

steps to install components in Single Node Openshift:

## Prerequisites

Download the Openshift client from the [official download page](https://access.redhat.com/downloads/content/290/ver=4.8/rhel---8/4.8.13/x86_64/product-software)

Download helm CLI: https://helm.sh/docs/intro/install/

## Step 1:

- Login to your SNO with oc CLI.

```
oc login -u kubeadmin -p [KUBEADMIN-PASSWORD] --server=https://api.[SNO.DOMAIN].com:6443
```

- Install helm template sno-pre-install.

This template will contain the operators of Openshift-GitOps, AMQ-Broker and the MachineConfig to mount HostPath for persistence of MongoDB and PostgreSQL.

```
helm install sno-pre-install ./sno-pre-install --create-namespace --namespace factory
```

Wait ~4 minutes until the SNO reboot caused by the MachineConfig.

## Step 2:

- Install helm template sno-install.

This template will contain the components required to deploy MongoDB, PostgreSQL and AMQ Broker.

```
helm install sno-install ./sno-install --namespace factory
```

Run the next command to check if the pods are running

```
oc get pods -n factory
```

If you can't see the broker pod "factory01-broker-ss-0" run the next command as workaround to restart the AMQ-Broker operator pod:

```
oc delete pod -l name=amq-broker-operator -n factory
```

Check again if the broker is created:
```
oc get pods -n factory
```

## Considerations

We need to provide persistent storage for databases, and for this context we add the easiest solution to integrate all-in-one providing hostPath storage which requires running pods as privileged, and SElinux chcon (done with MachineConfig CR), please note this is just for testing, in production environment you will need to add external storage to the NUC, like external disks, nfs, ODF, etc.