# **Dns-bind9**

## Prepare System
- Multiple VM Ubuntu version 20.04 on Hyper-V
- Example  [set up hyper-v on windows10]
- Update Package On Ubuntu 20.04
  ```sh
  sudo apt-get update
  ```
- Show hostname
  ```sh
  hostnamectl
  ```
- Set hostname
  ```sh
  sudo hostnamectl set-hostname dns.ake.com
  ```
- Show ip
  ```sh
  ifconfig
  ```
- Set ipv4 internal network (vEthernet-Internal-ME)
  - On cloud : you'll need to disable.
    ```sh
    sudo nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
    ```
    ```console
    network: {config: disabled}
    ```
  - Show file config in netplan
    ```sh
    ls /etc/netplan/
    ```
    ```console
    00-installer-config.yaml
    ```
  - Edit file config in netplan
    ```sh
    sudo nano /etc/netplan/00-installer-config.yaml
    ```
    ```console
    network:
      ethernets:
        eth0:
          dhcp4: false
          addresses:
            -  169.254.19.105/16
        eth1:
          dhcp4: true
      version: 2
    ```
  - Apply Config
    ```sh
    sudo netplan apply
    ```
----
<br/>

## Install bind9
- command
  ```sh
  sudo apt-get install bind9 bind9utils bind9-doc bind9-dnsutils bind9-host
  ```
- Check version bind9
  ```sh
  named -v
  ```
- Check version and build options bind9
  ```sh
  named -V
  ```
- Check Status Service
  ```sh
  systemctl status named
  ```
- Start Service
  ```sh
  sudo systemctl start named
  ```
- Enable auto start at boot time
  ```sh
  sudo systemctl enable named
  ```
- Get listens on TCP and UDP port 53
  ```sh
  sudo netstat -lnptu | grep named
  ```
- Check rndc status  
  check the status of the BIND name server
  ```sh
  sudo rndc status
  ```
----
<br/>

## Enable recursion service for outsite network
- Configuring the Options File <br/>
  edit file named.conf.options
    ```sh
    sudo nano /etc/bind/named.conf.options
    ```
    ```console
    acl "trusted" {
        127.0.0.1;         # localhost
        192.168.0.0/24;    # external network
        169.254.0.0/16;    # internal network
    };

    options {
            directory "/var/cache/bind";

            // If there is a firewall between you and nameservers you want
            // to talk to, you may need to fix the firewall to allow multiple
            // ports to talk.  See http://www.kb.cert.org/vuls/id/800113

            // If your ISP provided one or more IP addresses for stable
            // nameservers, you probably want to use them as forwarders.
            // Uncomment the following block, and insert the addresses replacing
            // the all-0's placeholder.

            // forwarders {
            //      8.8.8.8;
            //};

            //========================================================================
            // If BIND logs error messages about the root key being expired,
            // you will need to update your keys.  See https://www.isc.org/bind-keys
            //========================================================================
            dnssec-validation auto;

            listen-on { 169.254.19.105; };       #listen on private network only
            //listen-on-v6 { any; };

            // hide version number from clients for security reasons.
            version "not currently available";

            // optional - BIND default behavior is recursion
            recursion yes;

            // provide recursion service to trusted clients only
            allow-recursion { trusted; };

            allow-transfer { none; }; # disable zone transfers by default

            // enable the query log
            querylog yes;
    };
    ```
  check config named
    ```sh
    sudo named-checkconf
    ```
- Configuring the Local File <br/>
  edit File named.conf.local 
  ```sh
  sudo nano /etc/bind/named.conf.local
  ```
  ```console
  zone "ake.com" {
    type master;
    file "/etc/bind/zones/db.ake.com";       # zone file path
    //allow-transfer { 169.254.19.106; };    # ns2 private IP address - secondary
  };

  zone "254.169.in-addr.arpa" {
    type master;
    file "/etc/bind/zones/db.169.254";     # 169.254.0.0/16 subnet
    //allow-transfer { 169.254.19.106; };  # ns2 private IP address - secondary
  };
  ```
- Creating the Forward Zone File
  ```sh
  sudo mkdir /etc/bind/zones
  sudo cp /etc/bind/db.local /etc/bind/zones/db.ake.com
  sudo nano /etc/bind/zones/db.ake.com
  ```
  ```console
  ;
  ; BIND data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     dns.ake.com. admin.ake.com. (
                                3         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ;
  ; name servers - NS records
          IN      NS      dns.ake.com.

  ; name servers - A records
  dns.ake.com.            IN      A       169.254.19.105

  ; 169.254.0.0/16 - A records
  nexus.ake.com.          IN      A       169.254.19.101
  docker.ake.com.         IN      A       169.254.19.102
  rancher.ake.com.        IN      A       169.254.19.115
  jenkins.ake.com.        IN      A       169.254.19.103
  gitserver.ake.com.      IN      A       169.254.19.104
  ```
- Creating the Reverse Zone File
  ```sh
  sudo cp /etc/bind/db.127 /etc/bind/zones/db.169.254
  sudo nano /etc/bind/zones/db.169.254
  ```
  ```console
  ;
  ; BIND reverse data file for local loopback interface
  ;
  $TTL    604800
  @       IN      SOA     dns.ake.com. admin.ake.com. (
                                3         ; Serial
                           604800         ; Refresh
                            86400         ; Retry
                          2419200         ; Expire
                           604800 )       ; Negative Cache TTL
  ; name servers
          IN      NS      dns.ake.com.

  ; PTR Records
  105.19  IN      PTR     dns.ake.com.            ; 169.254.19.105
  101.19  IN      PTR     nexus.ake.com.          ; 169.254.19.101
  102.19  IN      PTR     docker.ake.com.         ; 169.254.19.102
  115.19  IN      PTR     rancher.ake.com.        ; 169.254.19.115
  103.19  IN      PTR     jenkins.ake.com.        ; 169.254.19.103
  104.19  IN      PTR     gitserver.ake.com.      ; 169.254.19.104
  ```
- Check config zone  
  check the "ake.com" forward zone configuration
  ```sh
  sudo named-checkzone ake.com /etc/bind/zones/db.ake.com
  ```
  check the "254.169.in-addr.arpa" reverse zone configuration
  ```sh
  sudo named-checkzone 254.169.in-addr.arpa /etc/bind/zones/db.169.254
  ```
- Restart Bind9
   ```sh
   sudo systemctl restart bind9
   ```
- Check firewall
  ```sh
  sudo ufw status
  ```
- Allow Port 53
  ```sh
  sudo ufw allow 53/tcp
  sudo ufw allow 53/udp
  ```
- Test Connect From Anathor VM
  ```sh
  dig A gitserver.ake.com @169.254.19.105
  ```
- View Log message of bind9
  ```sh
  sudo journalctl -eu named
  ```
----
<br/>

## How to Disable IPv6 in BIND
```sh
sudo nano /etc/default/named
```
```console
OPTIONS="-u bind -4"
```
```sh
sudo systemctl restart named
```

----
<br/>

## Setting the Default DNS Resolver on Ubuntu 20.04 Server (Client)
- The default recursive resolver can be seen with this command
  ```sh
  systemd-resolve --status
  ```
- As you can see, BIND isn’t the default. If you run the following command on the BIND server,
  ```sh
  dig A facebook.com
  ```
- Set BIND as the default resolver
  ```sh
  sudo nano /etc/systemd/resolved.conf
  ```
  ```console
  [Resolve]
  DNS=127.0.0.1
  #FallbackDNS=
  #Domains=
  #LLMNR=no
  #MulticastDNS=no
  #DNSSEC=no
  #DNSOverTLS=no
  #Cache=no-negative
  #DNSStubListener=yes
  #ReadEtcHosts=yes
  ```
  ```sh
  sudo systemctl restart systemd-resolved
  systemd-resolve --status
  ```

[set up hyper-v on windows10]: <https://github.com/EknarongAphiphutthikul/Hyper-V>