# Motivation
To test `www.example.org` behind ELB, the **development system** should see a
CNAME record pointing from `www.example.org` to
`www-123456789.eu-central-1.elb.amazonaws.com.`.

Unfortunately, dnsmasq on can only handle CNAME records for its own domain [^1].
But it can delegate a whole zone.

[CoreDNS](https://coredns.io/manual/toc/#setups) to the rescue!

[^1]: https://github.com/imp/dnsmasq/blob/master/dnsmasq.conf.example#L646


# Assumptions / Prerequisites
- NetworkManager on 127.0.0.1 contains dnsmasq, that can be configured via
  `/etc/NetworkManager/dnsmasq.d/` (for Ubuntu 18.04: see below).
- Docker and Docker Compose (see [NOTES.md](NOTES.md)) for manual installation


# CoreDNS
Run CoreDNS
```
docker-compose up -d
```

and validate that
```
dig +short @127.0.0.1 -p 1053 example.org
dig +short @127.0.0.1 -p 1053 foo.example.org
```
both return `127.0.0.1` as specified in `zones/example.org`.


# NetworkManager
Make NetworkManager use CoreDNS for `example.org`:
```
cat <<'EOF' | sudo tee -a /etc/NetworkManager/dnsmasq.d/example.org.conf
# this is coredns
server=/example.org/127.0.0.1#1053
EOF

sudo service NetworkManager reload

# to debug:
# journalctl -fu NetworkManager
```

Validate:
```
dig +short example.org
dig +short foo.example.org
```
should return the same lines as above.

Validate in browser (restart it first!):
```
python -mhttp.server 9876
```
- http://example.org:9876/
- http://foo.example.org:9876/


# Ubuntu 18.04
Disable resolved's stub resolver:
```
sudo -i
systemd --version  # >= 232 (https://unix.stackexchange.com/a/358485/120440)
echo 'DNSStubListener=no' >> /etc/systemd/resolved.conf
systemctl restart systemd-resolved
```

Enable dnsmasq in NetworkManager putting `dns=dnsmasq` into
`/etc/NetworkManager/NetworkManager.conf`, e.g.:
```
# /etc/NetworkManager/NetworkManager.conf
[main]
dns=dnsmasq
```

And restart NetworkManager:
```
systemctl restart NetworkManager
```
