# A6 Video Script — Network Performance Analysis (3–5 Minutes)

Record this using Zoom, Teams, OBS, or Panopto with screen share and microphone.

---

## Setup Before Recording

Connect to the EC2 instance using **EC2 Instance Connect** (browser terminal):
1. Go to **EC2 > Instances**, select your instance, click **Connect**.
2. Choose the **EC2 Instance Connect** tab (or **Session Manager** tab if that doesn't work).
3. Click **Connect** to open the browser-based terminal.

Make sure the following tools are installed before you start recording:

```bash
sudo dnf install traceroute nmap -y
```

Have the browser terminal full-screen and font large enough to be readable in the recording.

---

## Script

### [0:00–0:30] Introduction

**Say:**
> "Hello, my name is Charles Huff, and I am the cloud engineering consultant for Company Y. In this recording, I will perform a basic network performance analysis on the EC2 virtual machine that I configured as part of Company Y's cloud infrastructure. The purpose of this analysis is to verify that the VM is operating as expected and that network connectivity, latency, and service availability meet the requirements for a financial services web server."

**Show:** The browser-based EC2 Instance Connect terminal session.

**Run:**
```bash
date
hostname
curl -s http://169.254.169.254/latest/meta-data/instance-id
curl -s http://169.254.169.254/latest/meta-data/instance-type
```

**Say:**
> "Here we can see the current date and time, the hostname, the instance ID, and the instance type — confirming this is our t3.medium Amazon Linux 2023 instance."

---

### [0:30–1:30] Connectivity and Latency Test (ping)

**Run:**
```bash
ping -c 10 8.8.8.8
```

**Say:**
> "I am now running a ping test to Google's public DNS server at 8.8.8.8 to verify external network connectivity and measure round-trip latency. We are sending 10 ICMP echo request packets."

**After results appear, say:**
> "As we can see, all 10 packets were received with zero percent packet loss. The average round-trip time is approximately [X] milliseconds, which is well within acceptable ranges for a cloud-hosted server. This confirms that the VM has reliable outbound network connectivity."

---

### [1:30–2:30] Route Analysis (traceroute)

**Run:**
```bash
traceroute -m 15 8.8.8.8
```

**Say:**
> "Next, I am running a traceroute to the same destination to examine the network path from our EC2 instance. This shows us each network hop between our VM and the destination, along with the latency at each hop."

**After results appear, say:**
> "The traceroute completed in [X] hops. We can see the traffic passes through AWS's internal network infrastructure before reaching the public internet. The latency is consistent across hops, with no significant spikes or timeouts, indicating a healthy network path with no routing issues."

---

### [2:30–3:30] Service and Port Verification (ss and curl)

**Run:**
```bash
sudo ss -tlnp
```

**Say:**
> "Now I will verify that the expected services are listening on the correct ports using the `ss` command, which displays socket statistics. The `-t` flag shows TCP sockets, `-l` shows listening sockets, `-n` shows numeric port numbers, and `-p` shows the process owning each socket."

**After results appear, say:**
> "We can see that Apache HTTP Server (httpd) is listening on port 80. This is the expected service for our web server configuration. Administrative access is handled through EC2 Instance Connect and AWS Systems Manager Session Manager, which do not require open inbound ports. No unexpected services are listening, which confirms that our security configuration is correct."

**Run:**
```bash
curl -I http://localhost
```

**Say:**
> "I am also running a curl request to localhost to verify that the Apache web server responds correctly. The HTTP 200 status code and the 'Server: Apache' header confirm that the web server is functional and serving requests as expected."

---

### [3:30–4:15] Port Scan Verification (nmap)

**Run:**
```bash
sudo nmap -sT localhost
```

**Say:**
> "Finally, I am running an nmap TCP connect scan against localhost to verify the open ports from the instance's own perspective. This provides an additional confirmation of which services are accessible."

**After results appear, say:**
> "The nmap scan confirms that only port 80 (HTTP) is open on this instance, which matches our security group configuration and intended service profile. Administrative access is handled through AWS management-plane services rather than open inbound ports. There are no unexpected open ports that could represent a security risk for Company Y's infrastructure."

---

### [4:15–4:45] Summary and Conclusion

**Say:**
> "To summarize the findings of this network performance analysis:
>
> First, the ping test confirmed reliable external connectivity with zero packet loss and low latency of approximately [X] milliseconds.
>
> Second, the traceroute showed a clean network path with no routing anomalies or excessive hops.
>
> Third, the service verification confirmed that only the expected service — Apache on port 80 — is listening for inbound connections, with administrative access handled securely through AWS management services.
>
> Fourth, the nmap port scan verified that no unexpected ports are open.
>
> Based on this analysis, the VM is operating as expected and is ready to support Company Y's financial services workload. The network performance and security posture meet the requirements outlined in the infrastructure plan. Thank you."

---

## After Recording

- Save the recording as an `.mp4` file
- Name it something like `Charles_Huff_Network_Analysis.mp4`
- Submit alongside the report and screenshots PDF
