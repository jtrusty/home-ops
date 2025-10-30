# Fetch the leaf certificate
openssl s_client -connect 10.10.100.30:443 -servername localhost -showcerts </dev/null \
  2>/dev/null | awk '/BEGIN CERTIFICATE/,/END CERTIFICATE/' > ca.crt
