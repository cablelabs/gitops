# kind: MachineConfigPool

Common location for any default machine configs we want to apply to the cluster outside of specific operators

Any cluster specific machineconfigs should be include in the overlays/{{ cluster }}/machineconfigpool/ location

Example would be something specific for SRIOV
