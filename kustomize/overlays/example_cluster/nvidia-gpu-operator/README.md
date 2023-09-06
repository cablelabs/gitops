# GPU Operator

## Install operator
[Local operator install](../../../base/stable/nvidia-gpu-operator)

## Building Custom vGPU Containers

### vGPU Manager image

Download vGPU software from the [NVIDIA Licensing Portal](https://nvid.nvidia.com/dashboard/#/dashboard)

unzip the downloaded file
```bash
cd ~/Downloads
unzip NVIDIA-GRID-Linux-KVM-535.54.06-535.54.03-536.25.zip
```

clone the driver container image repository
```bash
cd ~/git
git clone https://gitlab.com/nvidia/container-images/driver
cd driver/vgpu-manager
cp -R rhel8 rhcos4.10
cd rhcos4.10
```

Copy the downloaded driver into the os directory
```bash
cp ~/Downloads/Host_Drivers/*-vgpu-kvm.run ./
```

Build image
```bash
export PRIVATE_REGISTRY=CHANGE-ME-REGISTRY-FQDN:4567/nvidia-team/nvidia-images VERSION=535.54.06 OS_TAG=rhcos4.10 CUDA_VERSION=12.2.0
podman image build \
  --build-arg DRIVER_VERSION=${VERSION} \
  --build-arg CUDA_VERSION=${CUDA_VERSION} \
  -t ${PRIVATE_REGISTRY}/vgpu-manager:${VERSION}-${OS_TAG} .
```

Upload image to private registry
```bash
podman push ${PRIVATE_REGISTRY}/vgpu-manager:${VERSION}-${OS_TAG}
```

### Custom vGPU Device Manager image

This was a container provided by a private NVIDIA repository.

Download container
```bash
podman pull nvcr.io/nvidia/cloud-native/vgpu-device-manager:v0.2.3-ubi8
```

Upload container to private registery
```
podman push $IMAGE CHANGE-ME-REGISTRY-FQDN:4567/nvidia-team/nvidia-images/vgpu-device-manager:v0.2.3-ubi8
```

### Private registery secret

Create a [secret](private-registry-secret.yaml) for private registry

## Cluster updates

Update [clusterpolicy.json](clusterpolicy.json) to enable GPU Passthrough and vGPU.

Add [labels](../custom-node-labels/custom-node-labels.yaml) to all workers to specify configuration.

Update [Kubevirt hyperconverged deployment](../openshift-cnv/kubevirt-hyperconverged-deployment.yaml) with your vGPU config and/or GPU passthrough devices.

## Add vGPU or passthrough GPU to a Virtual Machine

vGPU
```yaml
  gpus:
    - deviceName: nvidia.com/NVIDIA_L40-12Q
      name: GPU
      virtualGPUOptions:
        display:
          enabled: false
          ramFB:
            enabled: false
```

GPU Passthrough
```yaml
  gpus:
    - deviceName: nvidia.com/AD102GL_L40
      name: GPU
```
