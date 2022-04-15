# gitops
Example automation for managing and deploying a bare metal Kubernetes for cable and telecommunications workloads.

## Ansible

Ansible is used to provision the bare metal cluster then install ArgoCD for ongoing CI/CD operations.

## Kustomize

Kustomize is used as the configuration management solution for continued cluster operations. These artifacts are deployed via ArgoCD in a CI/CD pipeline.
