# How to Install & Configure Snort3 on Ubuntu22.04?

Why Snort3?

## Snort 2 vs Snort 3?

|         Feature          |                          Snort 2                          |                           Snort 3                           |
| :----------------------: | :-------------------------------------------------------: | :---------------------------------------------------------: |
|    **Packet threads**    |                      One per process                      |                   Any number per process                    |
|    **Config memory**     |                  use N processes * M GB                   |                M GB total, more for packets                 |
|    **Config reload**     |                    N processes, slower                    |       One thread that can be pinned to separate cores       |
|       **Plugins**        |              Limited to preprocs and outputs              |        Full plugin system with more than 200 plugins        |
|         **DAQ**          |                  2.X, run to completion                   |       3.X, vector input, multiple outstanding packets       |
|     **DAQ Modules**      |                    Only legacy modules                    |   Stacked modules, IOCTLs, file, socket and text modules    |
| **PCAP readback speed**  |                X Mbits/sec for Max-Detect                 |                2X with AC, 4X with hyperscan                |
|      **IP Layers**       |                          Two max                          |              Arbitrary and configurable limits              |
|    **IP reputation**     |                Complex with shared memory                 |                  Simplified process memory                  |
|      **Stream TCP**      |                  Complex implementation                   |               New and improved implementation               |
|  **Service detection**   |             AppID only, port configs required             |          Autodetection, most port configs optional          |
|    **HTTP inspector**    |                      Partly stateful                      |                       Fully stateful                        |
| **Port scan detection**  |           High, medium, and low thresholds only           |           Fully configurable detection thresholds           |
|    **Config parsing**    |                 Report one error and quit                 |                      Report all errors                      |
|     **Command line**     |             Some overlapping with config file             |    Set to override any config file from the command line    |
|    **Default config**    |                   Complex, needs tuning                   |                    Simplified, effective                    |
|   **Policy examples**    |                           None                            |          Tweaks to fit all standard Talos policies          |
|   **Policy bindings**    |                         One level                         |                           Nested                            |
|     **Rule syntax**      |          Inconsistent and requires line escapes           |          Uniform system with arbitrary whitespace           |
|     **Rule parsing**     |                Buggy with limited warnings                |           Robust with numerous optional warnings            |
|    **Rule comments**     |                       comments only                       |        #, #begin/#end marks, C-style and rem options        |
|   **Alert file rules**   |                            No                             |                             Yes                             |
| **Alert service rules**  |                            No                             |                             Yes                             |
| **Fast-pattern buffers** |                       Six available                       |                        14 available                         |
|   **SO rule features**   |                 Restricted functionality                  |                 True superset of text rules                 |
|   **Simple SO rules**    |                            No                             |                             Yes                             |
| **Dump built-in stubs**  |                    No (SO stubs only)                     |                             Yes                             |
|   **Runtime tracing**    |           No (debug tracing and misc logs only)           |                             Yes                             |
|    **Documentation**     |                 LaTeX-based PDF, READMEs                  |              ASCII docs, text, HTML, and PDFs               |
|     **Source code**      | 470,000 lines of C, with an average of 400 lines per file | 389,000 lines of C++, with an average of 200 lines per file |
|     **Distribution**     | Snort.org tarballs, with updates coming every six months  |      GitHub repo, with updates coming every two weeks       |



Snort 3 is significantly different from the Snort 2.9.9.x series. The configuration and rule files between the two versions are distinct and incompatible. Using the bundled `snort2lua` command, Snort 2 configuration and rule files may be converted to the Snort 3 format.



## How to Install Snort 3 on Ubuntu 22.04?

### 1. Update the Ubuntu Server

To ensure your Ubuntu 22.04 server is up-to-date and has the latest list of packages, run the following command:

```text
sudo apt-get update && sudo apt-get dist-upgrade -y
```



### 2. Install Dependencies

Prior to the build, a number of build tools and dependencies must be installed on Ubuntu 22.04 for a successful build and installation of Snort 3.

1. Run the following command, to install the package dependencies for Snort 3 installation on an Ubuntu 22.04 server:

   ```text
   sudo apt install build-essential libpcap-dev libpcre3-dev libnet1-dev zlib1g-dev luajit hwloc libdnet-dev libdumbnet-dev bison flex liblzma-dev openssl libssl-dev pkg-config libhwloc-dev cmake cpputest libsqlite3-dev uuid-dev libcmocka-dev libnetfilter-queue-dev libmnl-dev autotools-dev libluajit-5.1-dev libunwind-dev libfl-dev vim git -y
   ```

   

2. We will download a lot of source tarballs and other files, and we want to keep them in a single folder. Create a directory and change your location by running the following commands:

   ```text
   mkdir ~/snort_src && cd ~/snort_src
   ```

   

3. Snort DAQ (Data Acquisition library) is not included in the usual Ubuntu repositories. Thus, it must be compiled and installed from the source. Download and install the latest Snort DAQ by running the following commands:

   ```jsx
   git clone https://github.com/snort3/libdaq.git
   cd libdaq
   ./bootstrap
   ./configure
   make
   sudo make install
   ```

   

4. To download and install Google's `thread-caching malloc`, `Tcmalloc`, `a memory allocator` tuned for high concurrency conditions that will give a faster performance at the expense of increased memory consumption (this is a recommended but optional dependence), run the next commands:

   ```jsx
   cd ../
   wget https://github.com/gperftools/gperftools/releases/download/gperftools-2.9.1/gperftools-2.9.1.tar.gz
   tar xzf gperftools-2.9.1.tar.gz
   cd gperftools-2.9.1/
   ./configure
   make
   sudo make install
   ```

   

### 3. Download and Install Snort from Source Code

Now that all prerequisites have been met, we can download and install Snort 3 on Ubuntu 22.04. You can follow the steps given below for downloading and setting up the Snort 3.1.39.0, the most recent version at the time of writing.

1. To download the most recent version of the Snort tarball from the releases page, run the next commands:

   ```text
   cd..
   wget https://github.com/snort3/snort3/archive/refs/heads/master.zip
   unzip master.zip
   cd snort3-master
   ./configure_cmake.sh --prefix=/usr/local --enable-tcmalloc
   ```

   

   TIP

   If you are interested in enabling extra compile-time capabilities, such as the ability to handle large (over 2 GB) PCAP files or the new command line shell, run `./configure cmake.sh --help` to list all optional features, then attach them to the `./configure cmake.sh` command shown above.

   You will see similar output given below:

   ```jsx
   ------------------------------------------------------
   snort version 3.1.39.0
   Install options:
   prefix: /usr/local
   includes: /usr/local/include/snort
   plugins: /usr/local/lib/snort
   Compiler options:
   CC: /usr/bin/cc
   CXX: /usr/bin/c++
   CFLAGS: -fvisibility=hidden -DNDEBUG -g -ggdb -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -O2 -g -DNDEBUG
   CXXFLAGS: -fvisibility=hidden -DNDEBUG -g -ggdb -fno-builtin-malloc -fno-builtin-calloc -fno-builtin-realloc -fno-builtin-free -O2 -g -DNDEBUG
   EXE_LDFLAGS:
   MODULE_LDFLAGS:
   Feature options:
   DAQ Modules: Static (afpacket;bpf;dump;fst;gwlb;nfq;pcap;savefile;trace)
   libatomic: System-provided
   Hyperscan: OFF
   ICONV: ON
   Libunwind: ON
   LZMA: ON
   RPC DB: Built-in
   SafeC: OFF
   TCMalloc: ON
   JEMalloc: OFF
   UUID: ON
     -------------------------------------------------------
     -- Configuring done
     -- Generating done
     -- Build files have been written to: /home/alp/snort_src/snort3-master/build
   ```

   

2. To navigate to the build directory and install Snort 3 on Ubuntu 22.04 before compiling, run the following commands:

   ```text
   cd build
   make
   sudo make install
   ```

   

   This takes approximately half an hour, depending on your hardware. Snort is now installed under `/usr/local/`.

3. After the installation is completed, update shared libraries by running the next command. Otherwise, you get an error when you try to run Snort:

   ```text
   sudo ldconfig
   ```

   

4. To verify that Snort runs correctly, we pass the snort executable the -V flag:

   ```text
   snort -V
   ```

   

   You should see output similar to the following:

   ```jsx
   ,,_ -*> Snort++ <*-
   o" )~ Version 3.1.39.0
   '''' By Martin Roesch & The Snort Team
   http://snort.org/contact#team
   Copyright (C) 2014-2022 Cisco and/or its affiliates. All rights reserved.
   Copyright (C) 1998-2013 Sourcefire, Inc., et al.
   Using DAQ version 3.0.9
   Using LuaJIT version 2.1.0-beta3
   Using OpenSSL 3.0.2 15 Mar 2022
   Using libpcap version 1.10.1 (with TPACKET_V3)
   Using PCRE version 8.39 2016-06-14
   Using ZLIB version 1.2.11
   Using LZMA version 5.2.5
   ```

   

5. To test your Snort installation with the default configuration file, run the next command:

   ```text
   snort -c /usr/local/etc/snort/snort.lua
   ```

   

   You should see output similar to the following:

   ```jsx
   --------------------------------------------------
   o")~ Snort++ 3.1.39.0
   --------------------------------------------------
   Loading /usr/local/etc/snort/snort.lua:
   Loading snort_defaults.lua:
   Finished snort_defaults.lua:
   ssh
   hosts
   host_cache
   pop
   so_proxy
   stream_tcp
   mms
   smtp
   gtp_inspect
   packets
   dce_http_proxy
   stream_icmp
   normalizer
   ips
   network
   binder
   wizard
   appid
   file_id
   stream_udp
   http2_inspect
   http_inspect
   ftp_data
   search_engine
   ftp_server
   port_scan
   dce_http_server
   dce_smb
   dce_tcp
   netflow
   iec104
   cip
   telnet
   ssl
   sip
   rpc_decode
   modbus
   host_tracker
   stream_user
   stream_ip
   process
   back_orifice
   classifications
   dnp3
   active
   trace
   ftp_client
   decode
   alerts
   stream
   daq
   references
   arp_spoof
   output
   dns
   dce_udp
   imap
   file_policy
   s7commplus
   stream_file
   Finished /usr/local/etc/snort/snort.lua:
   Loading file_id.rules_file:
   Loading file_magic.rules:
   Finished file_magic.rules:
   Finished file_id.rules_file:
     --------------------------------------------------
   ips policies rule stats
   id loaded shared enabled file
   0 203 0 203 /usr/local/etc/snort/snort.lua
     --------------------------------------------------
   rule counts
   total rules loaded: 203
   text rules: 203
   option chains: 203
   chain headers: 1
     --------------------------------------------------
   service rule counts to-srv to-cli
   file_id: 203 203
   total: 203 203
     --------------------------------------------------
   fast pattern groups
   to_server: 1
   to_client: 1
     --------------------------------------------------
   search engine
   instances: 2
   patterns: 406
   pattern chars: 2436
   num states: 1724
   num match states: 364
   memory scale: KB
   total memory: 66.7676
   pattern memory: 18.2363
   match list memory: 26.5938
   transition memory: 21.6875
     --------------------------------------------------
   pcap DAQ configured to passive.
   Snort successfully validated the configuration (with 0 warnings).
     o")~ Snort exiting
   ```

   

6. Snort has several options to get more help. To view the Snort help options, run the next command:

   ```text
   snort --help
   ```

   

   To view the Snort command line option quick help, run the following command:

   ```text
   snort -?
   ```

   

   INFO

   Linux distributions based on Debian, such as Ubuntu 22.04, come pre-installed with the "apt" package manager. Essentially, it is a command-line-based utility used to perform app-related tasks such as installation, update, and uninstallation. Run the next commands to install Snort on Ubuntu 22.04 using `apt` manager:

   ```text
   sudo apt update
   sudo apt install snort -y
   ```

   

## How to Configure Snort 3 on Ubuntu 22.04

There are three configuration options for Snort: Sniffer mode, Packet logger mode, and Network IDS mode. We will set up Snort for Network IDS Mode in this section.

You can easily configure Snort 3 IPS software on your Ubuntu 22.04 server by following the 5 steps given in this section:

1. Configuring Network Interfaces
2. Configuring Snort Community and Local Rule Sets
3. Installing Snort OpenAppID
4. Configuring Snort Logging
5. Running Snort as a Service

### 1. Configuring Network Interfaces

Modern network cards employ offloading (LRO, for instance) to perform network packet reassembly in hardware, rather than software. In the majority of cases, this is desirable since it decreases system stress. For a NIDS, we must deactivate LRO and GRO, since they cause the truncation of longer packets. We must establish a systemD service in order to modify these values.

1. Determine the name(s) of the interface(s) on which Snort will listen by running the next command:

   ```text
   ip address show
   ```

   

   This will list your network interfaces on the Ubuntu server similar to below and we will select the WAN interface `ens18` to run Snort on, in our example:

   ```jsx
   1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
   link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
   inet 127.0.0.1/8 scope host lo
   valid_lft forever preferred_lft forever
   inet6 ::1/128 scope host
   valid_lft forever preferred_lft forever
   2: ens18: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
   link/ether 9e:d6:2e:8e:2c:2c brd ff:ff:ff:ff:ff:ff
   altname enp0s18
   inet 192.168.0.29/24 metric 100 brd 192.168.0.255 scope global dynamic ens18
   valid_lft 595176sec preferred_lft 595176sec
   inet6 fe80::9cd6:2eff:fe8e:2c2c/64 scope link
   valid_lft forever preferred_lft forever
     3: ens19: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
   link/ether 3a:49:6f:b5:60:20 brd ff:ff:ff:ff:ff:ff
   altname enp0s19
   inet 10.10.10.21/24 metric 100 brd 10.10.10.255 scope global dynamic ens19
   valid_lft 4776sec preferred_lft 4776sec
   inet6 fe80::3849:6fff:feb5:6020/64 scope link
   valid_lft forever preferred_lft forever
   ```

   

2. Set the interface on which Snort is listening for network traffic to promiscuous mode so that it may view all network traffic transmitted to it and not just traffic coming from the Ubuntu server by running the next command:

   ```jsx
   sudo ip link set dev ens18 promisc on
   ```

   

3. To verify that the network interface is running in promiscuous mode, run the following command:

   ```jsx
   ip address show ens18
   ```

   

   You should see the `PROMISC` pattern in the first line.

   ```jsx
   2: ens18: <BROADCAST,MULTICAST,PROMISC,UP,LOWER_UP> mtu 1500 qdisc mq state UP group default qlen 1000
   ```

   

4. Check the status of large-receive-offload (LRO) and generic-receive-offload (GRO) for those interfaces once you know the name of the network interface on which Snort will listen for traffic by running the following command (replacing `ens18` with your interface name):

   ```jsx
   ethtool -k ens18 | grep receive-offload
   ```

   

   You may see the following output:

   ```jsx
   generic-receive-offload: on
   large-receive-offload: off [fixed]
   ```

   

5. We need to ensure that both are set to `off` (or `off [fixed]`) if we see the output similar to the above. To disable Interface Offloading to prevent Snort from truncating large packets larger than 1518 bytes, run the following command (replacing `ens18` with your interface name):

   ```jsx
   sudo ethtool -K ens18 gro off lro off
   ```

   

6. Create and enable a systemd service unit to apply the modifications so that they survive after a system reboot by running the commands below:

   ```jsx
   sudo vim /etc/systemd/system/snort3-nic.service
   
   [Unit]
   Description=Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot
   After=network.target
   [Service]
   Type=oneshot
   ExecStart=/usr/sbin/ip link set dev ens18 promisc on
   ExecStart=/usr/sbin/ethtool -K ens18 gro off lro off
   TimeoutStartSec=0
   RemainAfterExit=yes
   [Install]
   WantedBy=default.target
   EOL
   ```

   

7. To reload systemd configuration settings, run the next command:

   ```jsx
   systemctl daemon-reload
   ```

   

8. Start and enable the service on boot, run the following command:

   ```text
   systemctl enable --now snort3-nic.service
   ```

   

9. To view the status of our newly created service run the following command:

   ```jsx
   service snort3-nic status
   ? snort3-nic.service - Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot
   Loaded: loaded (/etc/systemd/system/snort3-nic.service; enabled; vendor preset: enabled)
   Active: active (exited) since Mon 2022-08-22 12:19:02 UTC; 1min 39s ago
   Process: 41076 ExecStart=/usr/sbin/ip link set dev ens18 promisc on (code=exited, status=0/SUCCESS)
   Process: 41077 ExecStart=/usr/sbin/ethtool -K ens18 gro off lro off (code=exited, status=0/SUCCESS)
   Main PID: 41077 (code=exited, status=0/SUCCESS)
   CPU: 4ms
   Aug 22 12:19:02 snort systemd[1]: Starting Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot...
   Aug 22 12:19:02 snort systemd[1]: Finished Set Snort 3 NIC in promiscuous mode and Disable GRO, LRO on boot.
   ```

   

### 2. Configuring Snort Community and Local Rule Sets

Rulesets are the primary conduit for the Snort intrusion detection and prevention engine. Three kinds of Snort Rules exist:

- Community Rules
- Registered Rules
- Subscriber Rules

In this tutorial, the Community Snort rules and local will be installed.

1. Create some folders and files that Snort requires for rules by running the following commands:

   ```jsx
   sudo mkdir /usr/local/etc/rules
   sudo mkdir /usr/local/etc/so_rules/
   sudo mkdir /usr/local/etc/lists/
   sudo touch /usr/local/etc/rules/local.rules
   sudo touch /usr/local/etc/lists/default.blocklist
   sudo mkdir /var/log/snort
   ```

   

2. To add a rule which detects ICMP traffic and is excellent for validating if Snort is functioning properly and producing alarms, paste the following line into the `local.rules` file:

   ```jsx
   alert icmp any any -> any any ( msg:"ICMP Traffic Detected"; sid:10000001; metadata:policy security-ips alert; )
   ```

   

3. Start Snort and have it load the `local.rules` file (using the -R parameter) to ensure that these rules are loaded successfully by running the following command:

   ```jsx
   snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules
   ```

   

   The output should end with the following line:

   ```jsx
   Snort successfully validated the configuration (with 0 warnings).
   ```

   

   Scrolling through the results should reveal that this rule loaded correctly (under the rule counts section).

4. Run Snort in detection mode on an interface (replace eth0 with the name of your interface) and log all alarms to the console by entering the next command:

   ```jsx
   sudo snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules -i ens18 -A alert_fast -s 65535 -k none
   ```

   

   An explanation of the flags in the previous command is given below:

   - `-c`: /usr/local/etc/snort/snort.lua The snort.lua configuration file.
   - `-R`: /usr/local/etc/rules/local.rules The path to the rules file containing our one ICMP rule.
   - `-i ens18`: The interface to listen on.
   - `-A alert_fast`: Use the alert_fast output plugin to write alerts to the console.
   - `-s 65535`: Set the snaplen so Snort doesn't truncate and drop oversized packets.
   - `-k none`: Ignore bad checksums, otherwise, snort will drop packets with bad checksums

   Snort will load the configuration, then display the following lines:

   ```jsx
   Commencing packet processing
   ++ [0] ens18
   ```

   

   This indicates that Snort is presently checking all traffic on this interface, `ens18`, to the loaded rule. When traffic matches a rule, Snort will generate a console alert. Now, from another computer, ping to that interface's IP address of your Ubuntu server. Warnings should appear on the screen similar to the below:

   ```jsx
   08/22-12:45:11.714259 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] {ICMP} 192.168.0.29 -> 192.168.0.34
   08/22-12:45:13.455554 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/22-12:45:13.455606 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] {ICMP} 192.168.0.29 -> 192.168.0.34
   08/22-12:45:14.470965 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/22-12:45:14.471009 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] {ICMP} 192.168.0.29 -> 192.168.0.34
   ```

   

5. To stop Snort press `Ctrl-C`. This is a useful rule for testing Snort, but it may be too loud for production use, so you may comment it out with the hash symbol if you choose.

6. To enable decoder and inspector alerts (malicious traffic identified by Snort, not the rules owing to the rules' more complicated structure), and to notify the `ips` module where our rules file will be (due to the rules' more complex format), edit the `snort.lua` file:

   ```jsx
   sudo vi /usr/local/etc/snort/snort.lua
   ```

   

7. Look for the `ips` section on line 182, which is labeled `ips` and uncomment `enable_builtin_rules=true` (remove the preceding two dashes) and enable our local rules:

   ```jsx
   ips =
   {
   -- use this to enable decoder and inspector alerts
   enable_builtin_rules = true,
   include = RULE_PATH .. "/local.rules",
   -- use include for rules files; be sure to set your path
   -- note that rules files can include other rules files
   -- (see also related path vars at the top of snort_defaults.lua)
   variables = default_variables
   }
   ```

   

8. To verify the configuration of the Snort file, run the next command.

   ```jsx
   snort -c /usr/local/etc/snort/snort.lua
   ```

   

9. Run the Snort using the next command:

   ```jsx
   sudo snort -c /usr/local/etc/snort/snort.lua -i ens18 -A alert_fast -s 65535 -k none
   ```

   

   When you ping the Ubuntu Snort interface from another computer, the alarms should be rewritten to the console.

10. To download and install the Snort Community rules, run the following command

    ```jsx
    sudo su
    cd /usr/local/etc/rules && wget https://www.snort.org/downloads/community/snort3-community-rules.tar.gz
    tar xzf snort3-community-rules.tar.gz
    ```

    

11. To view the files, run the following command:

    ```jsx
    ls -l snort3-community-rules
    ```

    

    You should see similar to the below:

    ```jsx
    -rw-r--r-- 1 1210 root 7834 Aug 17 20:32 AUTHORS
    -rw-r--r-- 1 1210 root 15127 Aug 17 20:32 LICENSE
    -rw-r--r-- 1 1210 root 492722 Aug 17 20:32 sid-msg.map
    -rw-r--r-- 1 1210 root 1797899 Aug 17 20:32 snort3-community.rules
    -rw-r--r-- 1 1210 root 21084 Aug 17 20:32 VRT-License.txt
    ```

    

12. Edit the snort configuration file `/usr/local/etc/snort/snort.lua` with your favorite editor, like vi or nano.

    ```text
    vi /usr/local/etc/snort/snort.lua
    ```

    

13. Set the **HOME_NET** and **EXTERNAL_NET** environment variables. HOME_NET must be set to the networks that will be protected by Snort, such as the [WAN] IP address of your Ubuntu server, or the [LAN] network(subnets) behind the Ubuntu server. We will set this to the IP address of the Snort 3 `ens18` interface, 192.168.0.29/32 in our example. The EXTERNAL_NET is any network except our HOME_NET.

14. Go to `ips` block at line 182 and update the file as given below:

    ```jsx
    ips =
    {
    -- use this to enable decoder and inspector alerts
    enable_builtin_rules = true,
    include = RULE_PATH .. "/local.rules",
    include= RULE_PATH .. "/snort3-community-rules/snort3-community.rules"
    -- use include for rules files; be sure to set your path
    -- note that rules files can include other rules files
    -- (see also related path vars at the top of snort_defaults.lua)
    variables = default_variables
    }
    ```

    

15. Save and exit the configuration file.

16. Validate the configuration changes by running the following command:

    ```text
    snort -c /usr/local/etc/snort/snort.lua
    ```

    

    TIP

    Additionally, you may register on the Snort website. Registering enables you to obtain the Registered user rules using their Oink code. To obtain and install the Registered Snort rules you may follow the steps given below:

    1. Subscribe to the Snort website via `https://snort.org/users/sign_up`.

    2. Login to your Sort account and obtain the Oink code by navigating to the Oink code menu.

       ![Obtaining Snort Oinkcode](https://www.zenarmor.com/docs/assets/images/1-809dfee72beb5fa2873f672563dc9ce4.png) **Figure 1.** *Obtaining Snort Oinkcode*

    3. Click the `Downloads` link on the top navigation bar.

       ![Downloading Registered Snort V3.0 rule](https://www.zenarmor.com/docs/assets/images/2-2de3c82b61a648741a4f140285c74cc8.png) **Figure 2.** *Downloading Registered Snort V3.0 rule*

    4. Scroll down to the Rules and copy the link address of the latest Registered Snort V3.0 rule set, such as snortrules-snapshot-31350.tar.gz.

    5. To download the latest ruleset you may either run the next command(do not forget to replace the oinkcode pattern with your own code) or click on the latest Registered Snort V3.0 rule set in the download page then copy the file to your server:

       ```jsx
       wget https://www.snort.org/downloads/registered/snortrules-snapshot-31350.tar.gz?oinkcode=oinkcode -O snortrules-snapshot-31350.tar.gz
       ```

       

    6. To extract the registered rules run the following command:

       ```jsx
       sudo tar -xvf ~/snortrules-snapshot-31350.tar.gz -C /usr/local/etc/rules/
       ```

       

    7. View the registered rules by running the following command:

       ```jsx
       ls /usr/local/etc/rules/so_rules/
       ```

       

       You should see the similar output below:

       ```jsx
       browser-chrome.rules file-executable.rules file-office.rules malware-cnc.rules os-windows.rules protocol-other.rules rulestates-balanced-ips.states server-iis.rules server-webapp.rules
       browser-ie.rules file-flash.rules file-other.rules malware-other.rules policy-other.rules protocol-scada.rules rulestates-connectivity-ips.states server-mail.rules src
       browser-other.rules file-image.rules file-pdf.rules netbios.rules policy-social.rules protocol-snmp.rules rulestates-max-detect-ips.states server-mysql.rules
       browser-webkit.rules file-java.rules includes.rules os-linux.rules precompiled protocol-tftp.rules rulestates-no-rules-active.states server-oracle.rules
       exploit-kit.rules file-multimedia.rules indicator-shellcode.rules os-other.rules protocol-dns.rules protocol-voip.rules rulestates-security-ips.states server-other.rules
       ```

       

    8. The rule sets for registered users provide a substantial quantity of predefined detection rules that are helpful. If you have already tested Snort with the community rules, you may enable additional registered rules and then validate the configuration changes.

### 3. Installing Snort OpenAppID

OpenAppId is an application-centric detection language and processing module for Snort that allows users to build, distribute, and execute application and service detection. OpenAppID identifies application layer (layer 7) traffic and allows you to construct rules that act on application-layer traffic (for example, to prohibit Facebook or a certain Virtual Private Network type) and report traffic statistics for each identified kind of traffic. OpenAppID is an optional feature offered by Snort; you should activate it if you wish to detect or block certain kinds of traffic (FTP, Twitter, etc.) or gather statistics on the amount of data per type of traffic detected by your Snort server.

The Application Detector Package is a collection of detectors compiled by the Snort team in collaboration with the community and available for download and installation. You can activate the OpenAppID detectors (identifying traffic), then enable the OpenAppID metrics recording by following the steps given below:

1. Download and extract the OpenAppID detector package by running the next command:

   ```jsx
   cd ~/snort_src/
   wget https://snort.org/downloads/openappid/26425 -O OpenAppID-26425.tgz
   tar -xzvf OpenAppID-26425.tgz
   sudo cp -R odp /usr/local/lib/
   ```

   

   WARNING

   If you can't download the OpenAppID file from the URL given above, go to `https://snort.org/downloads` page and copy the link address of the `snort-openappid.tar.gz` under the OpenAppID heading.

2. Edit our Snort configuration file to point to this `odp` directory.

   ```text
   sudo nano /usr/local/etc/snort/snort.lua
   ```

   

3. Locate the `appid` section and configure it as follows:

   ```jsx
   appid =
   {
   -- appid requires this to use appids in rules
   app_detector_dir = '/usr/local/lib',
   log_stats = true,
   }
   ```

   

4. Validate the configuration changes by running the following command:

   ```text
   snort -c /usr/local/etc/snort/snort.lua
   ```

   

5. Modify Snort `local.rules` file with a new rule which will detect Facebook traffic by adding the next line:

   ```jsx
   alert tcp any any -> any any ( msg:"Facebook Detected"; appids:"Facebook"; sid:10000002; metadata:policy security-ips alert; )
   ```

   

6. Check the syntax of the `local.rules` file by running the following command:

   ```jsx
   snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules
   ```

   

7. Run Snort by executing the command below(by replacing `ens18` with your interface name):

   ```jsx
   snort -c /usr/local/etc/snort/snort.lua -R /usr/local/etc/rules/local.rules -i ens18 -A alert_fast -s 65535 -k none
   ```

   

8. Try to connect the Facebook site from your Ubuntu server. You should see the following alerts on the console:

   ```jsx
   08/23-10:56:06.262915 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 192.168.0.29:48824 -> 157.240.234.35:443
   08/23-10:56:06.263574 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 192.168.0.29:48824 -> 157.240.234.35:443
   08/23-10:56:06.265113 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 192.168.0.29:48824 -> 157.240.234.35:443
   08/23-10:56:06.261005 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 157.240.234.35:443 -> 192.168.0.29:48824
   08/23-10:56:06.265411 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 157.240.234.35:443 -> 192.168.0.29:48824
   08/23-10:56:06.266529 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 192.168.0.29:48824 -> 157.240.234.35:443
   ```

   

9. Press `Ctrl+C` to stop the Snort. You will see packet statistics similar to those given below:

   ```jsx
   Packet Statistics
     --------------------------------------------------
   daq
   received: 1034
   analyzed: 966
   dropped: 36
   outstanding: 68
   allow: 881
   whitelist: 85
   rx_bytes: 293090
     --------------------------------------------------
   codec
   total: 966 (100.000%)
   arp: 8 ( 0.828%)
   eth: 966 (100.000%)
   icmp4: 398 ( 41.201%)
   icmp6: 4 ( 0.414%)
   igmp: 4 ( 0.414%)
   ipv4: 954 ( 98.758%)
   ipv6: 4 ( 0.414%)
   ipv6_hop_opts: 2 ( 0.207%)
   tcp: 540 ( 55.901%)
   udp: 12 ( 1.242%)
     --------------------------------------------------
   Module Statistics
     -------------------------------------------------
   appid
   packets: 958
   processed_packets: 958
   total_sessions: 15
   service_cache_adds: 6
   bytes_in_use: 912
   items_in_use: 6
     --------------------------------------------------
   Appid Statistics
     --------------------------------------------------
   detected apps and services
   Application: Services Clients Users Payloads Misc Referred
   dns: 4 4 0 0 0 0
   facebook: 0 0 0 2 0 0
   http: 1 0 0 0 0 0
   putty: 0 1 0 0 0 0
   ssh: 1 0 0 0 0 0
   wget: 0 1 0 0 0 0
   https: 2 0 0 0 0 0
   ssl_client: 0 2 0 0 0 0
   ubuntu: 0 0 0 1 0 0
   icmp: 1 0 0 0 0 0
   icmp_for_ipv6: 2 0 0 0 1 0
   igmp: 2 0 0 0 0 0
   unknown: 1 0 0 0 0 0
     --------------------------------------------------
   Summary Statistics
     --------------------------------------------------
   process
   signals: 1
     --------------------------------------------------
   timing
   runtime: 00:03:23
   seconds: 203.166562
   pkts/sec: 5
   o")~ Snort exiting
   ```

   

### 4. Configuring Snort Logging

All output of events and packets is done by Loggers. To write Snort 3 events to log files, alert settings must be configured. There are several possibilities for logging in with Snort.

- *alert_csv:* output event in csv format
- *alert_ex:* output gid:sid:rev for alerts
- *alert_fast:* output event with brief text format
- *alert_full:* output event with full packet dump
- *alert_json:* output event in json format
- *alert_sfsocket:* output event over socket
- *alert_syslog:* output event to syslog
- *alert_unixsock:* output event over unix socket
- *log_codecs:* log protocols in packet by layer
- *log_hext:* output payload suitable for daq hext
- *log_pcap:* log packet in pcap format
- *Unified2:* output event and packet in uni?ed2 format file

1. To export event data to a file in short format (as stated in the command line above by option -A alert_type), go to the outputs section of the `snort.lua` configuration file.

   ```jsx
   ---------------------------------------------------------------------------
   -- 7. configure outputs
   ---------------------------------------------------------------------------
   -- event logging
   -- you can enable with defaults from the command line with -A <alert_type>
   -- uncomment below to set non-default configs
   --alert_csv = { }
   alert_fast = {file = true,
   packet = false,
   limit = 10,
   }
   --alert_full = { }
   --alert_sfsocket = { }
   --alert_syslog = { }
   --unified2 = { }
   -- packet logging
   -- you can enable with defaults from the command line with -L <log_type>
   --log_codecs = { }
   --log_hext = { }
   --log_pcap = { }
   -- additional logs
   --packet_capture = { }
   --file_log = { }
   ```

   

2. Save and exit the configuration file. The setting causes Snort to write logs to `alert_fast.txt` file. Check syntax by running the next command.

   ```jsx
   snort -c /usr/local/etc/snort/snort.lua
   ```

   

3. Rerun the Snort without the option, `-A alert_fast`. But specify the log directory with an option`-l /var/log/snort`.

4. Try to connect to the Facebook site from your Ubuntu server. You should see the following alerts in the `/var/log/snort/alert_fast.txt` file:

   ```jsx
   tail -f /var/log/snort/alert_fast.txt
   08/23-11:59:04.444640 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] [AppID: ICMP] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/23-11:59:04.444640 [**] [1:384:8] "PROTOCOL-ICMP PING" [**] [Classification: Misc activity] [Priority: 3] [AppID: ICMP] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/23-11:59:04.444686 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] [AppID: ICMP] {ICMP} 192.168.0.29 -> 192.168.0.34
   08/23-11:59:05.473735 [**] [1:382:11] "PROTOCOL-ICMP PING Windows" [**] [Classification: Misc activity] [Priority: 3] [AppID: ICMP] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/23-11:59:05.473735 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] [AppID: ICMP] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/23-11:59:05.473735 [**] [1:384:8] "PROTOCOL-ICMP PING" [**] [Classification: Misc activity] [Priority: 3] [AppID: ICMP] {ICMP} 192.168.0.34 -> 192.168.0.29
   08/23-11:59:05.473782 [**] [1:10000001:0] "ICMP Traffic Detected" [**] [Priority: 0] [AppID: ICMP] {ICMP} 192.168.0.29 -> 192.168.0.34
   08/23-11:59:05.618221 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 157.240.234.35:443 -> 192.168.0.29:48826
   08/23-11:59:05.618238 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 192.168.0.29:48826 -> 157.240.234.35:443
   08/23-11:59:05.618221 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 157.240.234.35:443 -> 192.168.0.29:48826
   08/23-11:59:05.618646 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 192.168.0.29:48826 -> 157.240.234.35:443
   08/23-11:59:05.618612 [**] [1:10000002:0] "Facebook Detected" [**] [Priority: 0] [AppID: Facebook] {TCP} 157.240.234.35:443 -> 192.168.0.29:48826
   ```

   

### 5. Running Snort as a Service

It is feasible to run Snort in the background as a daemon with the -D command line option, but it is also possible to establish a systemd service unit for Snort. To run Snort as a systemd service on your Ubuntu 22.04 server, you can follow the steps given below:

1. It is recommended to run Snort as a non-privileged system user if it is to be executed as a service. Create a system user account without a login for Snort:

   ```jsx
   useradd -r -s /usr/sbin/nologin -M -c SNORT_IDS snort
   ```

   

2. Grant the 'snort' user rights to the log directory by running the next commands:

   ```jsx
   sudo chmod -R 5775 /var/log/snort
   sudo chown -R snort:snort /var/log/snort
   ```

   

3. To create the Snort systemD service file, run the following command:

   ```jsx
   [Unit]
   Description=Snort3 NIDS Daemon
   After=syslog.target network.target
   [Service]
   Type=simple
   ExecStart=/usr/local/bin/snort -c /usr/local/etc/snort/snort.lua -s 65535 -k none -l /var/log/snort -D -u snort -g snort -i ens18 -m 0x1b --create-pidfile
   ExecStop=/bin/kill -9 $MAINPID
   [Install]
   WantedBy=multi-user.target
   ```

   

   Here is a summary of all the flags we use with Snort:

   - `/usr/local/bin/snort`: This is the path to the snort binary. We don't use `sudo` here since the script will be started with elevated (root) privileges.
   - `-c /usr/local/etc/snort/snort.lua`: The `snort.lua` configuration file.
   - `-s 65535`: Set the snaplen so Snort doesn't truncate and drop oversized packets.
   - `-k none`: Ignore bad checksums, otherwise, snort will drop packets with bad checksums, and they won't be evaluated.
   - `-l /var/log/snort`: The path to the folder where Snort will store all the log files it outputs.
   - `-D`: Run as a Daemon.
   - `-u snort`: After startup (and after doing anything that requires elevated privileges), switch to run as the "snort" user.
   - `-g snort`: After startup, run as the "snort" group.
   - `-i eth0`: The interface to listen on.
   - `-m 0x1b`: Umask of 033 for file permissions.
   - `--create-pidfile`: Create a PID file in the log directory

4. Enable the Snort systemD service and start it by running the next commands:

   ```jsx
   sudo systemctl enable snort3
   sudo service snort3 start
   ```

   

5. To view the Snort service status run the nexy command:

   ```jsx
   sudo service snort3 status
   ```

   

   Your output should be similar to the following,

   ```jsx
   ? snort3.service - Snort3 NIDS Daemon
   Loaded: loaded (/lib/systemd/system/snort3.service; enabled; vendor preset: enabled)
   Active: active (running) since Tue 2022-08-23 12:19:52 UTC; 4s ago
   Main PID: 1793 (snort)
   Tasks: 2 (limit: 9286)
   Memory: 232.7M
   CPU: 1.432s
   CGroup: /system.slice/snort3.service
   ??1793 /usr/local/bin/snort -c /usr/local/etc/snort/snort.lua -s 65535 -k none -l /var/log/snort -D -u snort -g snort -i ens18 -m 0x1b --create-pidfile
   Aug 23 12:19:53 snort snort[1793]: total memory: 2.64854
   Aug 23 12:19:53 snort snort[1793]: pattern memory: 0.420437
   Aug 23 12:19:53 snort snort[1793]: match list memory: 0.940338
   Aug 23 12:19:53 snort snort[1793]: transition memory: 1.25066
   Aug 23 12:19:53 snort snort[1793]: fast pattern only: 4963
   Aug 23 12:19:53 snort snort[1793]: --------------------------------------------------
   Aug 23 12:19:53 snort snort[1793]: pcap DAQ configured to passive.
   Aug 23 12:19:53 snort snort[1793]: Commencing packet processing
   Aug 23 12:19:53 snort snort[1793]: ++ [0] ens18
   Aug 23 12:19:53 snort snort[1793]: Writing PID "1793" to file "/var/log/snort/snort.pid"
   ```

   

6. Examine the whole output of the service using the following command, If there are issues:

   ```jsx
   sudo journalctl -u snort3.service
   ```