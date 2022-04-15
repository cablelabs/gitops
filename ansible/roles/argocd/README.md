# Install & Configure ArgoCD for OpenShift

Role to Install and Configure ArgoCD into an OpenShift Cluster

## Requirements

```bash
ansible-galaxy collection install kubernetes.core
```

## Required variables

Typically set in inventory

```init
argocd_git_password: Password for ArgoCD to access git
argocd_git_user: User for ArgoCD to access git
cluster_name: Name of the cluster
base_domain: cluster domain
argocd_git_repo: URL for the git server being linked to ArgoCD
argocd_repo_path: Path in git repo for ArgoCD to track
kubeconfig_certificate_authority_data: Kubeconfig certificate data
kubeconfig_context_user: kubeconfig user
kubeconfig_context_name: kubeconfig information
kubeconfig_client_certificate_data: client cert data
kubeconfig_client_key_data: client key data
```

## Example Playbook

```yaml
---
- name: Install ArgoCD
  hosts: localhost
  gather_facts: false

  tasks:
    - name: Install ArgoCD
      include_role:
        name: ../roles/argocd
```

## Syntax

```bash
ansible-playbook -vvv --ask-vault-pass -i inventories/{{ cluster_name }}/hosts playbooks/install_argocd.yml
```

## Things to do after install

* Add ArgoCD admin credentials to a secured password manager

```bash
export KUBECONFIG=$PATH_TO_FILE/kubeconfig
kubectl -n openshift-gitops get secret openshift-gitops-cluster -o jsonpath="{.data.admin\.password}" | base64 -d
```

* Update git overlays directory for your cluster with the applications/configurations you want to manage via CI/CD

## Version history

* 0.1 Initial Release
