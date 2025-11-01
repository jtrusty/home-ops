# kube-prometheus-stack

## TrueNAS Scale Apps

### graphite-exporter-mapping

```
mappings:
  ##########################################################################
  # SCALE namespaces you're receiving: scale.<profile>.(cgroup|net|system|…)
  ##########################################################################

  # --- CGROUPS -------------------------------------------------------------
  # CPU modes (user/system/…)
  - match: "scale.*.cgroup.*.cpu.*"
    name: "truenas_cgroup_cpu"
    help: "Per-cgroup CPU value by mode"
    labels:
      profile: "$1"
      cgroup:  "$2"
      mode:    "$4"   # user|system|idle|iowait|nice|…

  # CPU pressure (PSI): scale.<profile>.cgroup.<id>.cpu.<full|some>_pressure.<full|some>.<10|60|300>
  - match: "^scale\\.(.+)\\.cgroup\\.([^\\.]+)\\.cpu\\.(full|some)_pressure\\.(full|some)\\.(10|60|300)$"
    match_type: regex
    name: "truenas_cgroup_cpu_pressure"
    help: "Cgroup CPU pressure"
    labels:
      profile: "$1"
      cgroup:  "$2"
      kind:    "$3"
      level:   "$4"
      window:  "$5"

  # IO fields (read/write/…)
  - match: "scale.*.cgroup.*.io.*"
    name: "truenas_cgroup_io"
    help: "Per-cgroup IO field"
    labels:
      profile: "$1"
      cgroup:  "$2"
      field:   "$4"   # read|write|…

  # IO pressure (PSI)
  - match: "^scale\\.(.+)\\.cgroup\\.([^\\.]+)\\.io\\.(full|some)_pressure\\.(full|some)\\.(10|60|300)$"
    match_type: regex
    name: "truenas_cgroup_io_pressure"
    help: "Cgroup IO pressure"
    labels:
      profile: "$1"
      cgroup:  "$2"
      kind:    "$3"
      level:   "$4"
      window:  "$5"

  # Memory fields (anon, file, slab, kernel_stack, sock, anon_thp, …)
  - match: "scale.*.cgroup.*.mem.*"
    name: "truenas_cgroup_mem_bytes"
    help: "Per-cgroup memory field in bytes"
    labels:
      profile: "$1"
      cgroup:  "$2"
      field:   "$4"

  # Memory pressure (PSI)
  - match: "^scale\\.(.+)\\.cgroup\\.([^\\.]+)\\.mem\\.(full|some)_pressure\\.(full|some)\\.(10|60|300)$"
    match_type: regex
    name: "truenas_cgroup_mem_pressure"
    help: "Cgroup memory pressure"
    labels:
      profile: "$1"
      cgroup:  "$2"
      kind:    "$3"
      level:   "$4"
      window:  "$5"

  # --- SYSTEM / TEMPS ------------------------------------------------------
  # CPU package/core temperatures: scale.<profile>.cputemp.temperatures.<cpuX>
  - match: "scale.*.cputemp.temperatures.*"
    name: "truenas_cputemp_celsius"
    labels:
      profile: "$1"
      cpu:     "$3"   # cpu, cpu0, cpu1, …

  # System load: scale.<profile>.system.load.load{1,5,15}
  - match: "scale.*.system.load.*"
    name: "truenas_system_load"
    labels:
      profile: "$1"
      range:   "$3"   # load1|load5|load15

  # Uptime seconds: scale.<profile>.system.uptime.uptime
  - match: "scale.*.system.uptime.*"
    name: "truenas_system_uptime_seconds"
    labels:
      profile: "$1"

  # Clock sync flags/offsets:
  - match: "scale.*.system.clock_status.*"
    name: "truenas_clock_status"
    labels:
      profile: "$1"
      flag:    "$3"   # unsync|clockerr

  - match: "scale.*.system.clock_sync.*.*"
    name: "truenas_clock_sync"
    labels:
      profile: "$1"
      field:   "$3"   # offset|state

  # --- NETWORK -------------------------------------------------------------
  # Link carrier: scale.<profile>.net.carrier.<iface>.(up|down)
  - match: "scale.*.net.carrier.*.*"
    name: "truenas_net_carrier"
    labels:
      profile: "$1"
      iface:   "$3"
      state:   "$4"   # up|down

  # Duplex: scale.<profile>.net.duplex.<iface>.(full|half|unknown)
  - match: "scale.*.net.duplex.*.*"
    name: "truenas_net_duplex"
    labels:
      profile: "$1"
      iface:   "$3"
      mode:    "$4"

  # Operstate: scale.<profile>.net.operstate.<iface>.(up|down|…)
  - match: "scale.*.net.operstate.*.*"
    name: "truenas_net_operstate"
    labels:
      profile: "$1"
      iface:   "$3"
      state:   "$4"

  # Speed: scale.<profile>.net.speed.<iface>.speed  (bps)
  - match: "scale.*.net.speed.*.*"
    name: "truenas_net_speed_bps"
    labels:
      profile: "$1"
      iface:   "$3"

  # MTU: scale.<profile>.net.mtu.<iface>.mtu
  - match: "scale.*.net.mtu.*.*"
    name: "truenas_net_mtu_bytes"
    labels:
      profile: "$1"
      iface:   "$3"

  # Interface rx/tx (bytes/s style gauges from Netdata): scale.<profile>.net.<iface>.(received|sent)
  - match: "scale.*.net.*.*"
    name: "truenas_net_bytes"
    labels:
      profile: "$1"
      iface:   "$2"
      dir:     "$3"   # received|sent

  # Packets: scale.<profile>.net.packets.<iface>.(received|sent|multicast)
  - match: "scale.*.net.packets.*.*"
    name: "truenas_net_packets"
    labels:
      profile: "$1"
      iface:   "$3"
      field:   "$4"

  # Drops: scale.<profile>.net.drops.<iface>.(inbound|outbound)
  - match: "scale.*.net.drops.*.*"
    name: "truenas_net_drops"
    labels:
      profile: "$1"
      iface:   "$3"
      dir:     "$4"

  # --- SERVICES ------------------------------------------------------------
  # CPU per service: scale.<profile>.services.cpu.<svc>
  - match: "scale.*.services.cpu.*"
    name: "truenas_service_cpu_percent"
    labels:
      profile: "$1"
      service: "$3"

  # Memory per service: scale.<profile>.services.mem.usage.<svc>
  - match: "scale.*.services.mem.usage.*"
    name: "truenas_service_mem_mebibytes"
    labels:
      profile: "$1"
      service: "$4"

  # IO ops per service: scale.<profile>.services.io.ops.(read|write).<svc>
  - match: "scale.*.services.io.ops.*.*"
    name: "truenas_service_io_ops"
    labels:
      profile: "$1"
      op:      "$4"   # read|write
      service: "$5"

  # --- TrueNAS TRACING (arcstats, meminfo, cpu_usage, smart) ---------------
  # ARC stats: scale.<profile>.truenas.arcstats.<field>.<name>
  - match: "scale.*.truenas.arcstats.*.*"
    name: "truenas_arcstats_${3}"
    labels:
      profile: "$1"
      name:    "$4"   # ddhit, dmread, size, free, avail, …

  # Meminfo: scale.<profile>.truenas.meminfo.(available|total).<same>
  - match: "scale.*.truenas.meminfo.*.*"
    name: "truenas_meminfo_${3}_bytes"
    labels:
      profile: "$1"

  # CPU usage per CPU: scale.<profile>.truenas.cpu_usage.cpu.<cpuN>
  - match: "scale.*.truenas.cpu_usage.cpu.*"
    name: "truenas_cpu_usage_percent"
    labels:
      profile: "$1"
      cpu:     "$4"

  # SMART disk temps: scale.<profile>.smart_log.smart.disktemp.<serial>.<serial>
  - match: "scale.*.smart_log.smart.disktemp.*.*"
    name: "truenas_disktemp_celsius"
    labels:
      profile: "$1"
      disk:    "$4"   # serial or nvme0n1

  ##########################################################################
  # Your original Netdata/TrueNAS rules (servers.<host>....) — unchanged
  ##########################################################################

  - match: "servers.*.aggregation.*.*.*"
    help: "Aggregated CPU usage metrics (sum and avg) for TrueNAS"
    name: "truenas_aggregation_${3}"
    labels:
      host: "${1}"
      cpu: "${2}"
      cputime: "${4}"

  - match: "servers.*.cpu.*.*.*"
    help: "CPU usage metrics (user, system, idle, ...) for TrueNAS"
    name: "truenas_cpu_${3}"
    labels:
      host: "${1}"
      cpu: "${2}"
      cputime: "${4}"

  - match: "servers.*.cputemp.*.*"
    help: "CPU temperature for TrueNAS"
    name: "truenas_cputemp_${3}"
    labels:
      host: "${1}"
      cpu: "${2}"

  - match: "servers.*.ctl.*.*.*"
    name: "truenas_ctl_${3}"
    labels:
      host: "${1}"
      type: "${2}"
      id: "n/a"
      action: "${4}"

  - match: "servers.*.ctl.*.*.*.*"
    name: "truenas_ctl_${3}"
    labels:
      host: "${1}"
      type: "${2}"
      id: "${4}"
      action: "${5}"

  - match: "servers.*.df.*.*.*"
    help: "Disk / dataset usage for TrueNAS. How much storage is used / free"
    name: "truenas_df_${3}"
    labels:
      host: "${1}"
      dataset: "${2}"
      state: "${4}"

  - match: "servers.*.disk.*.*"
    help: "Disk I/O metrics for TrueNAS"
    name: "truenas_disk_${3}"
    labels:
      host: "${1}"
      disk: "${2}"

  - match: "servers.*.disk.*.*.*"
    help: "Disk I/O metrics for TrueNAS"
    name: "truenas_disk_${3}"
    labels:
      host: "${1}"
      disk: "${2}"
      action: "${4}"

  - match: "servers.*.geom_stat.*.*"
    help: "GEOM performance metrics"
    name: "truenas_geom_stat_${2}"
    labels:
      host: "${1}"
      disk: "${3}"

  - match: "servers.*.geom_stat.*.*.*"
    help: "GEOM performance metrics"
    name: "truenas_geom_stat_${2}"
    labels:
      host: "${1}"
      disk: "${3}"
      action: "${4}"

  - match: "servers.*.interface.*.*.*"
    help: "Network interface metrics for TrueNAS"
    name: "truenas_interface_${3}"
    labels:
      host: "${1}"
      interface: "${2}"
      action: "${4}"

  - match: "servers.*.load.*.*"
    help: "System load for TrueNAS"
    name: "truenas_load_${2}"
    labels:
      host: "${1}"
      range: "${3}"

  - match: "servers.*.memory.*.*"
    help: "Memory usage metrics for TrueNAS"
    name: "truenas_memory_${2}"
    labels:
      host: "${1}"
      state: "${3}"

  - match: "servers.*.nfsstat.*.*.*"
    help: "NFS statistics for TrueNAS"
    name: "truenas_nfsstat_${3}"
    labels:
      host: "${1}"
      role: "${2}"
      action: "${4}"

  - match: "servers.*.processes.*.*"
    help: "Number of processes (incl. states) in TrueNAS"
    name: "truenas_processes_${2}"
    labels:
      host: "${1}"
      state: "${3}"

  - match: "servers.*.rrdcached.*"
    name: "truenas_rrdcached_${2}"
    labels:
      host: "${1}"

  - match: "servers.*.rrdcached.*.*"
    name: "truenas_rrdcached_${2}"
    labels:
      host: "${1}"
      type: "${3}"

  - match: "servers.*.swap.*.*"
    name: "truenas_swap_${2}"
    help: "Swap space metrics for TrueNAS"
    labels:
      host: "${1}"
      state: "${3}"

  - match: "servers.*.uptime.*"
    name: "truenas_uptime_${2}"
    help: "Uptime metrics for TrueNAS"
    labels:
      host: "${1}"
      plugin: "uptime"

  - match: "servers.*.zfs_arc.*"
    name: "truenas_zfs_arc_${2}"
    help: "ZFS metrics (mainly cache) for TrueNAS"
    labels:
      host: "${1}"

  - match: "servers.*.zfs_arc.*.*"
    name: "truenas_zfs_arc_${2}"
    help: "ZFS metrics (mainly cache) for TrueNAS"
    labels:
      host: "${1}"
      type: "${3}"

  - match: "servers.*.zfs_arc.*.*.*"
    help: "ZFS metrics (mainly cache) for TrueNAS"
    name: "truenas_zfs_arc_${2}"
    labels:
      host: "${1}"
      type: "${3}"
      action: "${4}"

  - match: "servers.*.zfs_arc_v2.*.*"
    name: "truenas_zfs_arc_v2_${2}"
    labels:
      host: "${1}"
      plugin: "zfs_arc_v2"
      type: "${3}"
```
