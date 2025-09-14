FROM ubuntu:22.04

RUN apt-get update && apt-get install -y suricata iproute2 bind9-dnsutils

COPY suricata.yaml /etc/suricata/suricata.yaml
COPY rules/custom.rules /etc/suricata/rules/custom.rules

CMD ["sh", "-c", "sysctl -w net.ipv4.ip_forward=1 && suricata -c /etc/suricata/suricata.yaml -i eth0"]
