# 10 — Networking: Practice Questions

## Level 1 — Concepts

1. How many usable hosts are in a /26? A /20?
2. How does the kernel pick a route for a packet?
3. What is the role of `/etc/nsswitch.conf` in DNS lookups?
4. What does `0.0.0.0:8080` vs `127.0.0.1:8080` in `ss` output mean?
5. What do many `CLOSE_WAIT` sockets indicate? Many `TIME_WAIT`?
6. Why can `dig` and `getent hosts` return different answers?

<details><summary>Answers</summary>

1. /26 = 62. /20 = 4094.
2. Longest prefix match in the routing table. If no specific match, use the default route.
3. It sets lookup order: e.g., `files` (`/etc/hosts`) then `dns`.
4. `0.0.0.0` listens on all interfaces. `127.0.0.1` listens only locally.
5. CLOSE_WAIT: the local app does not close sockets after the peer closed (bug/leak). TIME_WAIT: normal after active close; high counts on busy clients/proxies.
6. `dig` queries DNS servers directly. `getent` follows nsswitch, including `/etc/hosts`.
</details>

## Level 2 — Commands

1. Show all IPs in brief form.
2. Show which interface and source IP are used to reach `10.20.0.5`.
3. List listening TCP/UDP ports with process names.
4. Test if TCP 5432 is open on `db.internal`.
5. Get only the A record for `example.com` from `1.1.1.1`.
6. Set a static IP `10.0.0.50/24`, gateway `10.0.0.1`, DNS `10.0.0.2` with nmcli.
7. Capture traffic to/from `10.0.0.9` on port 443 into a file.
8. Measure DNS, connect, TLS, and total time of an HTTPS request.
9. Add a persistent route to `172.20.0.0/16` via `10.0.0.1` (nmcli).
10. Check certificate expiry of `example.com`.

<details><summary>Answers</summary>

1. `ip -br addr`
2. `ip route get 10.20.0.5`
3. `ss -tulpn`
4. `nc -zv db.internal 5432`
5. `dig @1.1.1.1 +short A example.com`
6. `nmcli con mod eth0 ipv4.method manual ipv4.addresses 10.0.0.50/24 ipv4.gateway 10.0.0.1 ipv4.dns 10.0.0.2 && nmcli con up eth0`
7. `tcpdump -i any -nn host 10.0.0.9 and port 443 -w cap.pcap`
8. `curl -o /dev/null -s -w "dns:%{time_namelookup} conn:%{time_connect} tls:%{time_appconnect} total:%{time_total}\n" https://example.com`
9. `nmcli con mod eth0 +ipv4.routes "172.20.0.0/16 10.0.0.1" && nmcli con up eth0`
10. `openssl s_client -connect example.com:443 -servername example.com </dev/null 2>/dev/null | openssl x509 -noout -enddate`
</details>

## Level 3 — Scenarios

1. Users can `curl http://localhost:8080` on the server but not from outside. List causes.
2. `ping 8.8.8.8` works. `ping google.com` fails. What do you check?
3. An app's outbound calls are slow, only the first request. `curl -w` shows `time_namelookup` = 5 s. Cause?
4. In a K8s pod, resolving `api.example.com` creates 5+ DNS queries. Why? Fix?
5. A new EC2 instance cannot reach the internet. Walk through the checks.

<details><summary>Answers</summary>

1. App binds to `127.0.0.1`; host firewall (firewalld/ufw/iptables); cloud security group/NACL; SELinux port label; wrong port mapping (container).
2. DNS: `/etc/resolv.conf`, `resolvectl status`, `dig google.com`, `dig @1.1.1.1 google.com`, port 53 egress, systemd-resolved state.
3. DNS timeout on the first nameserver, then fallback (default timeout 5 s). Often IPv6 AAAA issues or a dead resolver. Fix resolver list, use `options timeout:1`, or `single-request-reopen`.
4. `ndots:5` and search domains: names with fewer than 5 dots are tried with each search suffix first. Use FQDN with trailing dot (`api.example.com.`) or lower `ndots` in `dnsConfig`.
5. Subnet route table has `0.0.0.0/0` to IGW (public) or NAT GW (private); public IP assigned; SG egress rules; NACL both directions (ephemeral ports); OS: `ip r` default route, DNS resolution, local firewall.
</details>

## Lab

1. Create two network namespaces linked by a veth pair. Assign IPs. Ping between them.
2. Run `python3 -m http.server 8080 --bind 127.0.0.1`. Try to reach it remotely. Rebind to `0.0.0.0`.
3. Capture a full TCP handshake with tcpdump and label SYN, SYN-ACK, ACK.
4. Use `netplan try` to apply a bad config and watch it roll back.
