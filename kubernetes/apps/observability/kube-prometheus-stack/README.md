# kube-prometheus-stack

## TrueNAS Scale Apps

### node-exporter

```
#!/usr/bin/env bash
set -euo pipefail

# === Config (you can tweak) ===================================================
SERVICE_NAME="node_exporter"
USER_NAME="nodeexp"
INSTALL_DIR="/usr/local/bin"
TEXTFILE_DIR="/var/lib/node_exporter/textfile_collector"
LISTEN_ADDR=":9100"

# Pass a version as first arg without the 'v' (e.g., 1.8.2). Default = latest.
REQUESTED_VER="${1:-latest}"

# === Helpers ==================================================================
die() { echo "ERROR: $*" >&2; exit 1; }

need_cmd() {
  command -v "$1" >/dev/null 2>&1 || die "Missing required command: $1"
}

arch_map() {
  local uarch
  uarch="$(uname -m)"
  case "$uarch" in
    x86_64)   echo "amd64" ;;
    aarch64)  echo "arm64" ;;
    armv7l)   echo "armv7" ;;   # unlikely for TrueNAS, included for completeness
    armv6l)   echo "armv6" ;;
    i386|i686) echo "386" ;;
    *) die "Unsupported architecture: $uarch" ;;
  esac
}

current_installed_version() {
  if [[ -x "${INSTALL_DIR}/${SERVICE_NAME}" ]]; then
    "${INSTALL_DIR}/${SERVICE_NAME}" --version 2>/dev/null | awk 'NR==1{print $3}' || true
  fi
}

latest_github_version() {
  # Returns version WITHOUT leading 'v'
  curl -fsSL "https://api.github.com/repos/prometheus/node_exporter/releases/latest" \
    | awk -F\" '/tag_name/ {print $4}' | sed 's/^v//'
}

download_and_verify() {
  local ver="$1" arch="$2" tmpdir="$3"
  local base="node_exporter-${ver}.linux-${arch}"
  local tgz="${base}.tar.gz"

  echo "→ Downloading node_exporter v${ver} (${arch})..."
  curl -fsSL -o "${tmpdir}/${tgz}" \
    "https://github.com/prometheus/node_exporter/releases/download/v${ver}/${tgz}"

  echo "→ Downloading checksums..."
  curl -fsSL -o "${tmpdir}/sha256sums.txt" \
    "https://github.com/prometheus/node_exporter/releases/download/v${ver}/sha256sums.txt"

  echo "→ Verifying checksum..."
  (cd "${tmpdir}" && grep " ${tgz}\$" sha256sums.txt | sha256sum -c -)

  echo "→ Extracting..."
  tar -xzf "${tmpdir}/${tgz}" -C "${tmpdir}"
  [[ -x "${tmpdir}/${base}/node_exporter" ]] || die "node_exporter binary not found after extract"
}

install_binary_if_changed() {
  local src_bin="$1" dest_bin="${INSTALL_DIR}/${SERVICE_NAME}"

  if [[ -x "$dest_bin" ]] && cmp -s "$src_bin" "$dest_bin"; then
    echo "✓ Binary unchanged; skipping replace."
    return 1
  fi

  echo "→ Installing binary to ${dest_bin}..."
  install -m 0755 "$src_bin" "$dest_bin"
  return 0
}

ensure_user_and_dirs() {
  echo "→ Ensuring user and directories..."
  if ! id -u "${USER_NAME}" >/dev/null 2>&1; then
    useradd --system --user-group --no-create-home --shell /usr/sbin/nologin "${USER_NAME}"
  fi
  mkdir -p "${TEXTFILE_DIR}"
  chown -R "${USER_NAME}:${USER_NAME}" "$(dirname "${TEXTFILE_DIR}")"
}

write_systemd_unit() {
  local unit="/etc/systemd/system/${SERVICE_NAME}.service"
  echo "→ Writing systemd unit: ${unit}"
  cat > "$unit" <<'UNIT'
[Unit]
Description=Prometheus Node Exporter
After=network.target

[Service]
User=nodeexp
Group=nodeexp
ExecStart=/usr/local/bin/node_exporter \
  --web.listen-address=:9100 \
  --collector.zfs \
  --collector.systemd \
  --collector.ethtool \
  --collector.conntrack \
  --collector.ipvs \
  --collector.interrupts \
  --collector.netclass \
  --collector.netdev \
  --collector.netstat \
  --collector.hwmon \
  --collector.thermal_zone \
  --collector.time \
  --collector.timex \
  --collector.edac \
  --collector.buddyinfo \
  --collector.pressure \
  --collector.sockstat \
  --collector.vmstat \
  --collector.udp_queues \
  --collector.textfile \
  --collector.textfile.directory=/var/lib/node_exporter/textfile_collector \
  --collector.filesystem \
  --collector.filesystem.mount-points-exclude=^/(proc|sys|dev|run|var/lib/containerd|var/lib/docker)($|/) \
  --collector.filesystem.fs-types-exclude=^(autofs|binfmt_misc|cgroup.*|debugfs|devpts|devtmpfs|fusectl|hugetlbfs|mqueue|proc|pstore|rpc_pipefs|securityfs|sysfs|tracefs)$
Restart=always
RestartSec=5
SyslogIdentifier=node_exorter

# If hwmon/ethtool need extra privileges on your hardware, you can add:
# AmbientCapabilities=CAP_NET_ADMIN CAP_SYS_RAWIO

[Install]
WantedBy=multi-user.target
UNIT
}

reload_enable_start() {
  systemctl daemon-reload
  systemctl enable --now "${SERVICE_NAME}"
}

restart_service() {
  systemctl restart "${SERVICE_NAME}"
}

verify_running() {
  echo "→ Checking service and version..."
  systemctl --no-pager --full status "${SERVICE_NAME}" || true
  curl -fsS "http://127.0.0.1${LISTEN_ADDR}/metrics" >/dev/null && echo "✓ Metrics endpoint responding on ${LISTEN_ADDR}" || echo "⚠ Could not reach metrics on ${LISTEN_ADDR}"
  echo -n "Installed version: "; "${INSTALL_DIR}/${SERVICE_NAME}" --version | awk 'NR==1{print $3}'
}

# === Main =====================================================================
need_cmd curl
need_cmd cmp
need_cmd tar
need_cmd sha256sum
need_cmd install
need_cmd systemctl
need_cmd awk
need_cmd sed

ARCH="$(arch_map)"

if [[ "${REQUESTED_VER}" == "latest" ]]; then
  echo "→ Resolving latest version from GitHub…"
  VER="$(latest_github_version)"
else
  VER="${REQUESTED_VER}"
fi
[[ -n "${VER}" ]] || die "Could not determine version"

CURR="$(current_installed_version || true)"
echo "Current installed: ${CURR:-none}"
echo "Target version:   ${VER}"

TMP="$(mktemp -d)"
trap 'rm -rf "$TMP"' EXIT

download_and_verify "${VER}" "${ARCH}" "${TMP}"

BIN_CHANGED=1
install_binary_if_changed "${TMP}/node_exporter-${VER}.linux-${ARCH}/node_exporter" || BIN_CHANGED=0

ensure_user_and_dirs
write_systemd_unit

if ! systemctl is-enabled --quiet "${SERVICE_NAME}" 2>/dev/null; then
  echo "→ Enabling and starting service..."
  reload_enable_start
else
  echo "→ Service already enabled."
  if [[ "${BIN_CHANGED}" -eq 1 ]]; then
    echo "→ Binary updated; restarting service..."
    restart_service
  else
    echo "→ No binary change; ensuring service is running..."
    systemctl start "${SERVICE_NAME}" || true
  fi
fi

verify_running
echo "Done."
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
