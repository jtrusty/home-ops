# kube-prometheus-stack

## TrueNAS Scale Apps

### node-exporter

```
command:
    --path.procfs=/host/proc
    --path.sysfs=/host/sys
    --collector.zfs
    --no-collector.filesystem

image: quay.io/prometheus/node-exporter:v1.9.1

network_mode: host

volumes:
    - /proc:/host/proc:ro
    - /sys:/host/sys:ro
```

### smartctl-exporter

```
command:
    --smartctl.interval=120s

image: quay.io/prometheuscommunity/smartctl-exporter:v0.14.0

privileged: True

user: root

volumes:
    - /dev:/dev:ro
    - /run/udev:/run/udev:ro
```
