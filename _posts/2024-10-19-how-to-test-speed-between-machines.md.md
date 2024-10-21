---
layout: post
title:  "How to Test Connection Speed Between Two Machines"
categories: linux unremarkable
excerpt_separator: <!--more-->
---

I want to access my cloud computes/volumes merge my networks together

<!--more-->

How to quickly test network speed can be a valuable skill. In this blog post, we'll walk through a simple way to test the network speed between two Linux servers using the `netcat` (GNU version), commonly referred to as `nc`. This lightweight utility is often used for networking tasks such as port scanning, data transfers, and service testing, but it can also be used for measuring throughput between systems.

## tl;dr

Do this:

1. On **'downloading' server** run `nc -l -p 12345 > /dev/null`
2. On **'uploading' server** run `dd if=/dev/zero bs=1M count=1000 | nc <server_B_IP> 12345`

That's it. You'll probably get something like `1048576000 bytes (1.0 GB) copied, 5.24369 s, 200 MB/s` printed on the 'uploading' server. If you don't, read on.

## Prerequisites

- Two Debian-based Linux servers (referred to here as **Server A** and **Server B**)
- Route between the two servers
- Netcat (GNU version) installed on both servers

## Step-by-Step Process

### 1. Install Netcat (if not already installed)

First, ensure that Netcat is installed on both servers. Most Linux distributions include it by default, but if not, you can install it using the package manager for your specific distribution.

For Debian/Ubuntu-based systems:

``` bash
sudo apt update
sudo apt install netcat
```

### 2. Start Netcat in Listening Mode on Server B

On **Server B** (the receiving server), we need to start Netcat in "listening mode" on a specific port. You can choose any unused port—here, we'll use port `12345` for illustration.

Run the following command on Server B:

``` bash
nc -l -p 12345 > /dev/null
```

**Explanation**

- `-l`: This tells Netcat to listen for incoming connections.
- `-p 12345`: Specifies port 12345.
- `> /dev/null`: Discards the received data. We are not interested in the actual data transfer but in testing the speed, so we send the data to `/dev/null`.

### 3. Send Data from Server A

On **Server A** (the sending server), we can now send data to Server B. To measure throughput, we will use the `dd` command to generate data and pipe it to Netcat, which will send the data to Server B.

Run the following command on Server A:

``` bash
dd if=/dev/zero bs=1M count=1000 | nc <server_B_IP> 12345
```

**Explanation**

- `dd if=/dev/zero`: This generates a stream of null bytes from `/dev/zero`.
- `bs=1M`: This sets the block size to 1 megabyte.
- `count=1000`: This specifies the number of blocks to send. Here, we’re sending 1000 MB (1 GB) of data.
- `| nc <server_B_IP> 12345`: This pipes the data to Netcat, which sends it to the IP address of Server B on port 12345.

### 4. Measure Transfer Speed

As the data is transferred from Server A to Server B, the `dd` command will output statistics, including the total time taken and the speed at which the data was transmitted.

You will see an output similar to this on Server A:

``` bash
1000+0 records in
1000+0 records out
1048576000 bytes (1.0 GB) copied, 5.24369 s, 200 MB/s
```

In this example, the network speed between the two servers is approximately **200 MB/s**.

## Interpreting Results

The output from the `dd` command will provide a good indication of the network speed between Server A and Server B. However, keep in mind that factors such as disk I/O performance, CPU load, and network congestion can also affect the speed measurements. For more accurate results, consider running the test multiple times and averaging the results.

## Additional Considerations

- Routing: Ensure the route between machines is as expected.
- Firewall Rules: Ensure that the port you're using (e.g., 12345) is open on both servers. If necessary, modify the firewall rules to allow traffic through this port.
- Bandwidth: The measured throughput is the raw data transfer speed between the two servers. Depending on the link and conditions, real-world application performance may vary.
- Compression: If you're sending compressible data over the network, the transfer rate might be higher than expected due to compression during transmission. Confirm with data generated from `/dev/random`.

## Conclusion

Testing network speed between two Linux servers using Netcat is a quick and easy process. While tools like `iperf` and `nload` are commonly used for similar tests, Netcat provides a lightweight, no-frills alternative that works just as well in many cases. By combining it with the `dd` command, you can generate traffic and measure the speed between servers, helping you diagnose network bottlenecks or performance issues in your infrastructure.

Feel free to experiment with different data sizes and block sizes to get a more comprehensive view of your network performance!
