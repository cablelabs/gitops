# MetalLB in BGP mode

In BGP mode, each node in the cluster establishes a BGP peering session with the network router
and uses the session to advertise the IPs of cluster services.

To configure BGP on MetalLB, we need the following information:

- router IP address
- router AS number
- MetalLB AS number
- IP address range

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    peers:
    - peer-address: 10.10.10.1
      peer-asn: 64512
      my-asn: 64512
    address-pools:
    - name: default
      protocol: bgp
      addresses:
      - 10.10.10.10-10.10.10.98
      avoid-buggy-ips: true
```

After the BGP peering session is established, check the speaker logs for BGP advertisement messages.

- Example

```output
{"caller":"bgp_controller.go:232","event":"updatedAdvertisements","ip":"10.10.10.12","msg":"making advertisements using BGP","numAds":1,"pool":"default","protocol":"bgp","service":"argocd/argocd-server","ts":"2021-04-29T22:25:21.49316365Z"}
{"caller":"main.go:343","event":"serviceAnnounced","ip":"10.10.10.12","msg":"service has IP, announcing","pool":"default","protocol":"bgp","service":"argocd/argocd-server","ts":"2021-04-29T22:25:21.493196292Z"}
```

## BGP configuration on Mellanox switch

- On the switch, make sure the following global configuration exists:
  - Disable spanning-tree
  - Enable LLDP
  - Enable IP routing (L3)
  - Enable BGP protocol

    ```commands
     switch(config)# no spanning-tree
     switch(config)# lldp
     switch(config)# ip routing
     switch(config)# protocol bgp```

- Configure the AS number

```commands
switch(config)# router bgp 64512
```

- Configure the BGP neighbors, in this case, the MetalLB speakers.

```commands
switch(config router bgp 64512)# neighbor 10.10.10.117 remote-as 64512
switch(config router bgp 64512)# neighbor 10.10.10.116 remote-as 64512
.
.
.
```

- Verify the BGP configuration

```commands
show ip bgp summary
show ip bgp neighbors
```

- Example
  - Before peering is established

    ```output
     BGP neighbor: 10.10.10.117, remote AS: 64512, link: internal
     BGP version: 4, remote router ID: 0.0.0.0
     BGP State: ACTIVE
     Last read: 0:00:00:00, last write: 0:00:00:00, hold time is: 0, keepalive interval in seconds: 0
     Configured hold time in seconds: 180, keepalive interval in seconds: 60
     Minimum holdtime from neighbor in seconds: 0```

  - After peering is established (MetalLB is connected and advertises its own router ID)

  ```output
     BGP neighbor: 10.10.10.117, remote AS: 64512, link: internal
     BGP version: 4, remote router ID: 10.10.10.117
     BGP State: ESTABLISHED
     Last read: 0:00:00:05, last write: 0:00:00:05, hold time is: 90, keepalive interval in seconds: 30
     Configured hold time in seconds: 180, keepalive interval in seconds: 60
     Minimum holdtime from neighbor in seconds: 90
  ```

- To check the advertised service IPs from MetalLB

```commands
show ip route bgp
```
