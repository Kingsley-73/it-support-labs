# Network Connectivity & Linux Service Troubleshooting Lab

**Status:** Completed — simulated personal lab

## 1. Project Objective

Practice a structured troubleshooting workflow on an Ubuntu Linux virtual machine by checking network configuration, gateway reachability, external IP connectivity, hostname resolution, SSH service status, and authentication logs.

## 2. Environment

- **Operating system:** Ubuntu Linux
- **Environment:** Local VirtualBox virtual machine
- **Network interface:** `enp0s8`
- **IPv4 address:** `10.0.2.15/24`
- **Default gateway:** `10.0.2.2`
- **External connectivity test:** `1.1.1.1`
- **Hostname resolution test:** `ubuntu.com`
- **SSH test target:** `127.0.0.1` (localhost)

## 3. Investigation Steps

1. Inspected the network interface and IPv4 configuration using `ip -br addr`.
2. Reviewed the routing table using `ip route`.
3. Tested gateway reachability using `ping -c 4 10.0.2.2`.
4. Tested external IP connectivity using `ping -c 4 1.1.1.1`.
5. Checked hostname resolution using `getent hosts ubuntu.com`.
6. Reviewed authentication logs in `/var/log/auth.log`.
7. Checked the SSH service and socket states using `systemctl`.
8. Examined SSH-related system logs and performed a local SSH connection test.

## 4. Findings

- **Network interface:** `enp0s8` was up and had the IPv4 address `10.0.2.15/24`.
- **Routing:** A default route was present through `10.0.2.2`.
- **Gateway connectivity:** Four packets were received with no packet loss.
- **External connectivity:** Four ICMP replies were received from `1.1.1.1`, with no packet loss during the test.
- **Hostname resolution:** `ubuntu.com` resolved to IPv6 addresses.
- **SSH service:** The initial status showed `ssh.service` inactive while `ssh.socket` was listening. A subsequent local SSH connection activated the SSH service, and the login succeeded.
- **Authentication logs:** The inspected logs included local administrative activity. These observations alone did not establish an unauthorized access attempt.

## 5. Troubleshooting Interpretation

The successful gateway and external IP tests indicated basic network reachability at the time of testing. Successful hostname resolution indicated that name resolution was functioning for the tested hostname.

These checks do not prove that every application, port, or external destination is reachable. ICMP tests do not verify TCP application services, and a successful DNS lookup does not guarantee that a website or service is available.

The SSH service/socket behavior demonstrated socket activation: systemd can listen on the SSH socket and start the SSH service when a connection arrives. The successful local SSH login confirmed that the local connection worked in the tested configuration.

## 6. Limitations

- Testing was performed in a local virtual machine, not a production network.
- Connectivity results represent the time of testing.
- Only selected destinations and services were checked.
- The available authentication-log evidence was insufficient to conclude that a security incident had occurred.
- The local SSH test did not establish external SSH accessibility.

## 7. Skills Demonstrated

- Linux command-line troubleshooting
- Network interface and routing inspection
- Basic connectivity testing
- DNS resolution checks
- SSH service and socket diagnostics
- Authentication-log review
- Evidence-based technical documentation

## 8. Conclusion

This lab provided practical experience following a repeatable troubleshooting sequence, interpreting command output, distinguishing successful checks from their limitations, and documenting findings without making unsupported security conclusions.

**Project type:** Personal learning lab  
**Environment:** Local Ubuntu VM  
**Evidence:** Screenshots captured during the lab session
