---
layout: post
title:  "Solving Docker Swarm “No Route to Host” Error: A Step-by-Step Guide"
categories: linux oracle-cloud firewall unremarkable
---

While setting up a Docker Swarm cluster, I encountered a frustrating “No route to host” error when trying to join a Docker Swarm node to the manager. The root cause was a set of firewall rules that were blocking traffic between the nodes.

---

In this blog post, I'll walk through the steps I followed to resolve this issue, including diagnosing the problem, modifying `iptables` rules, and ensuring the necessary packages and tools were installed. Here’s how I solved the problem, step by step.

## Problem

When attempting to join a Docker Swarm node to the manager using the following command:

``` bash
docker swarm join --token <token> 10.0.0.242:2377
```

I encountered the following error:

``` bash
Error response from daemon: rpc error: code = Unavailable desc = connection error: desc = "transport: Error while dialing: dial tcp 10.0.0.242:2377: connect: no route to host"
```

This error indicated that traffic on port `2377` between the nodes was being blocked, preventing the worker node from joining the Swarm.

---

## Diagnosing the Issue

To diagnose the issue, I began by checking the connectivity between the two nodes (`10.0.0.234` and `10.0.0.242`) using the following tools:

### 1. Ping

I ran a `ping` test to ensure basic network connectivity:

``` bash
ping 10.0.0.242
```

This succeeded, meaning the nodes could see each other on the network. However, the problem was specific to Docker Swarm’s management port (`2377`).

### 2. Netcat (`nc`)

To test if port `2377` was reachable, I used `netcat` (`nc`) to check if the port was open:

``` bash
nc -vz 10.0.0.242 2377
```

This failed with an error, indicating that traffic on port `2377` was blocked.

---

## Installing Required Tools

Since neither `telnet` nor `nc` (netcat) were installed on the system, I needed to install the necessary packages for testing port connectivity.

### Install `netcat`

``` bash
sudo apt update
sudo apt install netcat
```

---

## Investigating Firewall (`iptables`)

The next step was to check if the firewall (configured via `iptables`) was blocking the necessary ports for Docker Swarm communication.

### Checking `iptables` Rules

I listed the current `iptables` rules on the manager node (`10.0.0.242`):

``` bash
sudo iptables -L INPUT -v --line-numbers
```

This revealed the following rule set:

``` bash
Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
num   pkts bytes target     prot opt in     out     source               destination         
1     3092  895K ACCEPT     all  --  any    any     anywhere             anywhere             state RELATED,ESTABLISHED
2        2   168 ACCEPT     icmp --  any    any     anywhere             anywhere            
3      178 17942 ACCEPT     all  --  lo     any     anywhere             anywhere            
4        0     0 ACCEPT     udp  --  any    any     anywhere             anywhere             udp spt:ntp
5       67  3956 ACCEPT     tcp  --  any    any     anywhere             anywhere             state NEW tcp dpt:ssh
6       31  3397 REJECT     all  --  any    any     anywhere             anywhere             reject-with icmp-host-prohibited
```

The key issue was the `REJECT` rule at line 6, which rejected all traffic that wasn’t explicitly allowed by the previous rules. Since there was no rule allowing traffic on port `2377`, it was being blocked.

---

## Why Docker Swarm Ports Are Blocked

To understand why Docker Swarm ports were being blocked, let's look at the existing rules in detail:

- **Rule 1 (Allow Related and Established Traffic)**:  
  This rule allows all **related and established** connections. This means that responses to outgoing connections and already established communication are allowed. However, new incoming connections (such as new Docker Swarm traffic on ports `2377`, `7946`, and `4789`) are not allowed by this rule unless they are part of an existing connection.

``` bash
ACCEPT all -- anywhere anywhere state RELATED,ESTABLISHED
```

- **Rule 2 (ICMP Traffic)**:  
  This rule allows **ICMP** traffic (e.g., pings) between nodes. ICMP is used for diagnostic tools like `ping`, but it doesn't affect other types of communication like Docker Swarm, which uses TCP and UDP. This is why `ping` works between the nodes, but Docker Swarm communication fails.

``` bash
ACCEPT icmp -- anywhere anywhere
```

- **Rule 3 (Loopback Interface)**:  
  This rule allows all traffic on the **loopback** interface (`lo`), which is the internal interface of the machine. This rule ensures that services communicating internally on the host can function correctly. However, it doesn't impact external traffic between different nodes, such as Docker Swarm communication.

``` bash
ACCEPT all -- lo any anywhere anywhere
```

- **Rule 4 (UDP for NTP)**:  
  This rule allows **UDP** traffic specifically for NTP (Network Time Protocol) and doesn't apply to any other UDP traffic. Since Docker Swarm requires other UDP ports (e.g., 4789 and 7946), this rule doesn't affect them, and these ports remain blocked.

``` bash
ACCEPT udp -- anywhere anywhere udp spt:ntp
```

- **Rule 5 (TCP for SSH)**:  
  This rule allows **TCP** traffic specifically for SSH (port `22`) and doesn't affect other TCP traffic, such as Docker Swarm traffic on port `2377`. Since it only applies to SSH (`tcp dpt:ssh`), Docker Swarm traffic remains blocked by later rules unless explicitly allowed.

``` bash
ACCEPT tcp -- anywhere anywhere state NEW tcp dpt:ssh
```

- **Rule 6 (REJECT all other traffic)**:  
  This rule is a catch-all that **rejects all traffic not explicitly allowed by earlier rules**. Since there is no earlier rule that allows Docker Swarm's required traffic (ports `2377`, `7946`, `4789`), this rule effectively blocks all other communication.

``` bash
REJECT all -- anywhere anywhere reject-with icmp-host-prohibited
```

Since the existing rules only allow related/established traffic, ICMP, loopback, NTP, and SSH, and there are no rules to allow Docker Swarm's required ports, **rule 6 blocks all other communication, including Docker Swarm traffic**.

## Allowing Docker Swarm Ports

[Docker Swarm requires the following ports to be open](https://docs.docker.com/engine/swarm/swarm-tutorial/#open-protocols-and-ports-between-the-hosts) for node communication between managers and workers:

- **TCP 2377**: Swarm management traffic
- **TCP/UDP 7946**: Gossip and data plane traffic between nodes
- **UDP 4789**: VXLAN overlay network traffic

These ports should be opened on **both the Swarm manager and Swarm worker nodes**. This ensures that communication flows correctly between all nodes in the cluster.

To allow the necessary traffic for Docker Swarm, I inserted an `ACCEPT` rule for port `2377` before the `REJECT` rule on both the manager and worker:

``` bash
sudo iptables -I INPUT 6 -p tcp --dport 2377 -j ACCEPT
```

Additionally, I opened ports `7946` and `4789` for Docker Swarm communication:

``` bash
sudo iptables -I INPUT 6 -p tcp --dport 7946 -j ACCEPT
sudo iptables -I INPUT 6 -p udp --dport 7946 -j ACCEPT
sudo iptables -I INPUT 6 -p udp --dport 4789 -j ACCEPT
```

These commands allow both TCP and UDP traffic on the ports required for Docker Swarm to function correctly. After adding these rules, I verified them:

``` bash
sudo iptables -L INPUT -v --line-numbers
```

The output showed that traffic on ports `2377`, `7946`, and `4789` was now allowed before the `REJECT` rule.

---

## Verifying Connectivity

With the firewall updated, I tested the port connectivity again using `netcat`:

``` bash
nc -vz 10.0.0.242 2377
```

This time, the connection succeeded, confirming that port `2377` was open and reachable.

I repeated this for ports `7946` and `4789` to ensure they were open and reachable as well:

``` bash
nc -vz 10.0.0.242 7946
nc -vz 10.0.0.242 4789
```

All ports responded successfully.

---

## Saving the `iptables` Rules

After updating your `iptables` rules to allow Docker Swarm traffic, it's important to save the changes to ensure they persist after a reboot or network restart. By default, `iptables` rules are stored in memory, so they will be lost unless explicitly saved.

### How to Save the `iptables` Rules

To save the `iptables` rules permanently, you can use the following command, which uses `tee` to write the rules to a file while requiring `sudo` for the protected directory:

``` bash
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

This command works as follows:

- `sudo iptables-save` exports the current `iptables` rules.
- The output is then passed to `sudo tee /etc/iptables/rules.v4`, which writes the rules to `/etc/iptables/rules.v4` (the standard location for IPv4 rules on most Linux systems).
  
This will ensure that the rules are saved and can be restored automatically when the system reboots.

If you're working with IPv6 rules, you can similarly save them using:

``` bash
sudo ip6tables-save | sudo tee /etc/iptables/rules.v6
```

## Successfully Joining Docker Swarm

After confirming that all necessary ports were reachable, I was able to successfully join the worker node to the Docker Swarm cluster:

``` bash
docker swarm join --token <token> 10.0.0.242:2377
```

The node joined the Swarm without any errors.

---

## Conclusion

This issue was caused by firewall rules blocking traffic on Docker Swarm’s management port (`2377`) and other required ports. By diagnosing the problem with `ping` and `netcat`, and then updating `iptables` to allow the necessary traffic, I was able to resolve the issue. Here’s a quick recap of the steps:

1. **Diagnose**: Use `ping` and `netcat` to test basic network connectivity and port availability.
2. **Install Tools**: Ensure `netcat` or `telnet` are installed for testing ports.
3. **Check Firewall**: Use `iptables` to check for rules blocking Docker Swarm ports.
4. **Open Required Ports on Both Nodes**: Open Docker Swarm ports (`2377`, `7946`, `4789`) in the firewall on both the manager and worker nodes.
5. **Test and Join**: Reattempt Docker Swarm join after confirming ports are open.

With these steps, you can troubleshoot and resolve similar Docker Swarm communication issues.

---

## Useful Commands Recap

- **Check Connectivity**: `nc -vz <IP> <port>`
- **View Firewall Rules**: `sudo iptables -L INPUT -v --line-numbers`
- **Allow Port in Firewall**: `sudo iptables -I INPUT <num> -p tcp --dport <port> -j ACCEPT`
- **Restart Docker Swarm**: `docker swarm init --advertise-addr <IP>`

---

Feel free to leave a comment if you encounter any issues or have additional questions!
