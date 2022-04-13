# Performance Profiles (PAO)

- Install the pao operator
- Create profiles for nodes and place them in the profiles directory
-- Example profiles located in overlays/defaults/pao-4.8/profiles

## Enablement

Once the profile is created make sure to label any nodes you wish to use that profile with the correct MachineConfigPools label.

```example
Change from
'node-role.kubernetes.io/worker'
to
'node-role.kubernetes.io/worker-sriov'
```

## Current uses

Current the PAO profile can be used for the following

- Kernel Args
- CPU isolation - How to determine which cpu's are which example below.

```bash
lscpu|grep -i NUMA
NUMA node(s):        2
NUMA node0 CPU(s):   0-23,48-71
NUMA node1 CPU(s):   24-47,72-95
```

To determine sibling to match it with run the following

```bash
cat /sys/devices/system/cpu/cpu0/topology/thread_siblings_list
0,48
```

This tells you CPU0 matches with CPU48.

```example breakout
You could then send 0-1,48-49,24-25,72-73 as reserved(systemd usage) and the rest as isolated 
```

- Hugepages
- Enablement of real time kernel

Sysctl settings are managed via tuned not PAO, systemd, udev rules, etc.. would be managed by a MachineConig.

## Upstream Link

[Link to Red Hat documentation](https://docs.openshift.com/container-platform/4.8/scalability_and_performance/cnf-performance-addon-operator-for-low-latency-nodes.html)
