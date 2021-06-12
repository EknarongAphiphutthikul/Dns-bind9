# **Dns-bind9**

## Prepare System
- Multiple VM Ubuntu version 20.04 on Hyper-V
- Example  [set up hyper-v on windows10]
## Update Package On Ubuntu 20.04
```sh
sudo apt-get update
```
## Show hostname
```sh
hostnamectl
```
## Set hostname
```sh
hostnamectl set-hostname dns.ake.com
```
## Show ip
```sh
ifconfig
```
## Set ipv4 internal network (vEthernet-Internal-ME)
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
## Install bind9
```sh
sudo apt-get install bind9 bind9utils bind9-doc bind9-dnsutils bind9-host
```
## Check version bind9
```sh
named -v
```
## Check version and build options bind9
```sh
named -V
```
## Check Status Service
```sh
systemctl status named
```
## Start Service
```sh
sudo systemctl start named
```
## Enable auto start at boot time
```sh
sudo systemctl enable named
```
## Get listens on TCP and UDP port 53
```sh
sudo netstat -lnptu | grep named
```
## Check rndc status
check the status of the BIND name server
```sh
sudo rndc status
```
## Enable recursion service for outsite network
- edit file named.conf.options
    ```sh
    sudo nano /etc/bind/named.conf.options
    ```
    ```console
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

        listen-on-v6 { any; };

        // hide version number from clients for security reasons.
        version "not currently available";

        // optional - BIND default behavior is recursion
        recursion yes;

        // provide recursion service to trusted clients only
        allow-recursion { 127.0.0.1; 192.168.0.0/24; 169.254.0.0/16; };

        // enable the query log
        querylog yes;
    };
    ```
-  Check Config
    ```sh
    sudo named-checkconf
    ```
- Restart Bind9
   ```sh
   sudo systemctl restart named
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
  dig A google.com @169.254.19.105
  ```
- View Log message of bind9
  ```sh
  sudo journalctl -eu named
  ```

[set up hyper-v on windows10]: <https://github.com/EknarongAphiphutthikul/Hyper-V>