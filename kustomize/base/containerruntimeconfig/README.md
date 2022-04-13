# ContainerRuntimeConfig guidelines - Limit 10 per cluster

## Defaults we are deploying

1. pidsLimit: 2048 (1024 is the defauit for OpenShift)
2. logLevel: info
3. overlaySize: 10G
4. logSizeMax: "-1"

If you want to customize these defaults for each cluster copy the overlay-size.yml from this directory and include it in the cluster overlays directory and then make your changes. Remember to stop including this base directory so you don't get conflicts.

## Upstream documentation

[Link to Red Hat documentation](https://docs.openshift.com/container-platform/4.8/post_installation_configuration/machine-configuration-tasks.html)
