# SR-IOV Network Configuration

- Install the OpenShift SRIOV Network Operator.
  Create the openshift-sriov-network-operator namespace, add an OperatorGroup CR and subscribe to the SR-IOV Network Operator using _k8s/base/sriov/sriov-operator-install.yml_.
- Configure the SR-IOV Network Operator using _sriov-operator-config.yml_.
  The Operator Admission Controller webhook daemon set, _enableOperatorWebhook_ is set to false to overcome an issue with adding a SriovNetworkNodePolicy for an unsupported NIC.
- Create a configMap for the unsupported NIC using _unsupported-nic-config.yml_ that specified the Vendor ID, PF Device ID, VF Device ID.

**Note**: To determine the PF Device ID, login to one of the SRIOV-enabled nodes and run the command ``lspci -nn | grep Eth``. In this example, the Vendor ID is 8086 and PF Device ID is 1563.

```example
07:00.0 Ethernet controller [0200]: Intel Corporation Ethernet Controller 10G X550T [8086:1563] (rev 01)
```

To determine the VF Device ID, create SR-IOV Virtual Functions(VFs) manually with ``echo 8 > /sys/class/net/enp7s0f1/device/sriov_numvfs`` and get the VF ID (example: 1565) from ``lspci -nn | grep Eth``.

```example
 07:10.1 Ethernet controller [0200]: Intel Corporation X550 Virtual Function [8086:1565]
```

- Specify the SR-IOV network device configuration for a node by creating a SriovNetworkNodePolicy object. Add the nodeSelector label, the vendor/deviceID/pfNames for the nicSelector along with the desired number of VFs to be created and a unique resource name.

## Determine udev kernel parameters to enable sriov on reboot per network device

- Log into a worker node and determine udev information

```example
udevadm info -a /sys/class/net/ens6f0
```

That will provide you with the data needed for the the 99-sriov.rules file.

```bash
cat /etc/udev/rules.d/99-sriov.rules
KERNELS=="0000:a1:00.0", SUBSYSTEM=="pci", DRIVERS=="mlx5_core", ATTR{vendor}=="0x15b3", ATTR{device}=="0x101d", ATTR{sriov_numvfs}="8" 
```

- Update the 99-worker-sriov.yml with the information for your worker node.

```bash
cat << EOF | base64
KERNELS=="0000:a1:00.0", SUBSYSTEM=="pci", DRIVERS=="mlx5_core", ATTR{vendor}=="0x15b3", ATTR{device}=="0x101d", ATTR{sriov_numvfs}="8"
EOF
```

This provides output like the below

```example
S0VSTkVMUz09IjAwMDA6YTE6MDAuMCIsIFNVQlNZU1RFTT09InBjaSIsIERSSVZFUlM9PSJtbHg1
X2NvcmUiLCBBVFRSe3ZlbmRvcn09PSIweDE1YjMiLCBBVFRSe2RldmljZX09PSIweDEwMWQiLCBB
VFRSe3NyaW92X251bXZmc309IjgiCg==
```

Use that to update the contents source: in 99-sriov.rules

### Make sure PAO is enabled with a profile configured for SRIOV

### Update Node labels and remove the standard worker label and replace it with SRIOV labels

This can be done in the custom-node-labels directory in overlays for each cluster.

```example
node-role.kubernetes.io/worker-sriov: ''
feature.node.kubernetes.io/network-sriov.capable: 'true'
```

### Verify SRIOV configuration

Please refer to the documentation in k8s/examples/sriov to test the SRIOV configuration.
