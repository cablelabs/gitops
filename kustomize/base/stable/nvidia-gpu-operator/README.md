# NVIDIA GPU Operator

NVIDIA GPU Operator

## Requirements

NFD-Operator Installed

## Installation

Followed instructions from [NVIDIA](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/openshift/introduction.html#openshift-introduction)

## Notes

Openshift < 4.9.9 we need to create a machineconfig per cluster with the correct entitlements which will install the required kernel modules. [Entitlement Instructions](https://docs.nvidia.com/datacenter/cloud-native/gpu-operator/openshift/cluster-entitlement.html#cluster-entitlement)

Add this to the k8s/overlays/{{ cluster }} directory as it will be unique per installation.

A generic Cluster policy has been included in the defaults directory. Copy this to the cluster overlay to be included, incase we need any customization later.
