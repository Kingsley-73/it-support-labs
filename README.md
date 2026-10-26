# Network Connectivity & Linux Service Troubleshooting Lab

**Status:** Completed — simulated personal lab.

## Objective
Practice a repeatable troubleshooting workflow: inspect interface and routes, test gateway reachability, test external IP connectivity, verify hostname lookup, and inspect SSH service/socket state and authentication logs.

## Environment
- Ubuntu Linux VM in a local VirtualBox lab
- Interface: `enp0s8`
- VM IPv4 address: `10.0.2.15/24`
- Default gateway: `10.0.2.2`
- External ICMP target: `1.1.1.1`
- Hostname lookup: `ubuntu.com`
- SSH test target: `127.0.0.1` (loopback on the same VM)

These are observations from the captured lab session, not production-network claims.

## Workflow and findings

### 1. Inspect interface and routing
```bash
ip -br addr
ip route
ip -4 addr show enp0s8
```
The interface was up with `10.0.2.15/24`, and the routing table showed a default route via `10.0.2.2`. This shows local configuration was present; it does not alone prove end-to-end connectivity.

### 2. Test the gateway
```bash
ping -c 4 10.0.2.2
```
**Result:** 4 packets transmitted, 4 received, 0% loss. The gateway responded to ICMP during the test.

### 3. Test an external IP
```bash
ping -c 4 1.1.1.1
```
**Result:** 4 packets transmitted, 4 received, 0% loss; average RTT was about 164.5 ms. This supports basic external IP reachability at that moment, but does not prove all destinations or application ports are available.

### 4. Check hostname lookup
```bash
getent hosts ubuntu.com
```
**Result:** IPv6 address records were returned. This supports successful name lookup for this hostname, but does not prove every DNS query type or HTTPS application works.

### 5. Review authentication logs
```bash
sudo tail -n 50 /var/log/auth.log
sudo grep -Ei 'sshd.*(Failed password|Invalid user|Accepted password|Accepted publickey)' /var/log/auth.log | tail -n 20
```
The captured output included `sudo` and cron entries. The displayed filtered lines did **not** clearly confirm repeated SSH login failures. The simulated incident title alone is not proof an attack occurred.

### 6. Check SSH service and socket
```bash
sudo systemctl status ssh --no-pager
sudo systemctl status ssh.socket --no-pager
sudo journalctl -u ssh --since "30 minutes ago" --no-pager
sudo journalctl -u ssh.socket --since "30 minutes ago" --no-pager
```
Observed in the screenshots:
- Initially, `ssh.service` was inactive while `ssh.socket` was active/listening on port 22.
- The SSH service journal later showed the service starting and listening on port 22, followed by a local connection timing out before authentication.
- A subsequent `ssh kingsley@127.0.0.1` connection succeeded and displayed the Ubuntu login banner.

This is consistent with socket activation in this environment. Exact behavior depends on system configuration.

## Results summary

| Check | Result | Conclusion |
|---|---|---|
| Interface and route | Interface up; default route present | Local configuration was present |
| Gateway ping | 4/4 replies | Gateway reachable by ICMP |
| External IP ping | 4/4 replies | External IP reachable by ICMP at test time |
| Hostname lookup | Address records returned | Name lookup succeeded for this hostname |
| SSH log review | No clear repeated failures in the captured filtered output | Repeated SSH failures were not confirmed by this evidence |
| SSH service/socket | Socket listening; service later started; local login succeeded | SSH was available for a later local connection |

## Likely next step if an application still failed
With gateway ping, external ICMP, and hostname lookup succeeding, test the **specific application and port** rather than repeating pings. For HTTPS:
```bash
curl -Iv --connect-timeout 10 https://ubuntu.com
```
Review the TCP connection, TLS handshake, and HTTP response. If it fails, use the error to guide checks of proxy, firewall, TLS, or application configuration.

## Limitations
- This is a point-in-time lab snapshot, not continuous monitoring.
- ICMP can be blocked or deprioritized even when an application works; successful ping does not prove TCP/443 or another port is open.
- `getent hosts` confirms a name-service result, not a complete DNS health assessment.
- The screenshots contain partial logs; an event missing from this excerpt may exist elsewhere.
- SSH was tested against loopback, not a remote host, so remote firewall/network access was not validated.
- No packet capture or full root-cause investigation is included.

## Evidence
Screenshots are stored in [`evidence/`](evidence/). They support the observations above; the report distinguishes observed results from conclusions that cannot be proven from the captures.

## Safety and scope
All activity was performed in a local simulated lab. No production systems were targeted. Do not commit credentials, passwords, or private keys to this repository.
