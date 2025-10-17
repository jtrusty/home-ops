# Cilium

## UniFi BGP

```sh
router bgp 64513
  bgp router-id 192.168.1.1
  no bgp ebgp-requires-policy

  neighbor talos-clus01 peer-group
  neighbor talos-clus01 remote-as 64514

  neighbor 10.10.16.141 peer-group talos-clus01
  neighbor 10.10.16.142 peer-group talos-clus01
  neighbor 10.10.16.143 peer-group talos-clus01
  neighbor 10.10.16.144 peer-group talos-clus01
  neighbor 10.10.16.145 peer-group talos-clus01
  neighbor 10.10.16.146 peer-group talos-clus01

  address-family ipv4 unicast
    neighbor k8s next-hop-self
    neighbor k8s soft-reconfiguration inbound
  exit-address-family
exit
```