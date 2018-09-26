# Doing Things Manually
Install coredns binary:
```
cd /tmp
wget https://github.com/coredns/coredns/releases/download/v1.2.2/release.coredns_1.2.2_linux_amd64.tgz
tar xf release.coredns_1.2.2_linux_amd64.tgz
sudo mv coredns /usr/local/bin/
coredns --help
```

Run:
```
coredns -dns.port=1053
```


# DNS request flow for `felix.example.org` (with CNAME):

- browser asks dnsmasq on `127.0.1.1:53/udp`
- dnsmasq asks CoreDNS on `127.0.1.1:1053/udp`
- CoreDNS
  - prepares an answer containing the CNAME
  - asks Google (`8.8.8.8`) for the load balancer's A-records
  - answers with both CNAME and A-records
