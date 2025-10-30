# Cilium

## UniFi BGP

```sh
ip prefix-list K8S-LB seq 5 permit 10.10.16.150/29 le 32

route-map K8S-LB-IN permit 10
  match ip address prefix-list K8S-LB
route-map DENY-ALL-OUT deny 10

router bgp 64513
  bgp router-id 192.168.1.1
  no bgp ebgp-requires-policy
  bgp bestpath as-path multipath-relax

  neighbor talos-clus01 peer-group
  neighbor talos-clus01 remote-as 64514
  neighbor talos-clus01 timers 3 9

  neighbor 10.10.16.141 peer-group talos-clus01
  neighbor 10.10.16.142 peer-group talos-clus01
  neighbor 10.10.16.143 peer-group talos-clus01
  neighbor 10.10.16.144 peer-group talos-clus01
  neighbor 10.10.16.145 peer-group talos-clus01
  neighbor 10.10.16.146 peer-group talos-clus01

  address-family ipv4 unicast
    neighbor talos-clus01 activate
    neighbor talos-clus01 next-hop-self
    neighbor talos-clus01 soft-reconfiguration inbound
    neighbor talos-clus01 route-map K8S-LB-IN in
    neighbor talos-clus01 route-map DENY-ALL-OUT out
    neighbor talos-clus01 maximum-prefix 16 90 warning-only
  exit-address-family
exit
```

## Cilium CRDs v2
```
# BGPv2 CRDs (apiVersion: cilium.io/v2)
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.18.3/pkg/k8s/apis/cilium.io/client/crds/v2/ciliumbgpadvertisements.yaml
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.18.3/pkg/k8s/apis/cilium.io/client/crds/v2/ciliumbgppeerconfigs.yaml
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.18.3/pkg/k8s/apis/cilium.io/client/crds/v2/ciliumbgpclusterconfigs.yaml
kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/1.18.3/pkg/k8s/apis/cilium.io/client/crds/v2/ciliumloadbalancerippools.yaml
```