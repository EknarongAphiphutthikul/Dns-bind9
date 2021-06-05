# **Dns-bind9**

## Prepare System
    Multiple VM Ubuntu version 20.04 on Hyper-V
    Example : https://github.com/EknarongAphiphutthikul/Hyper-V/blob/main/script.txt

## Update Package On Ubuntu 20.04
```sh
sudo apt-get update
```
## show hostname
```sh
hostnamectl
```
## set hostname
```sh
hostnamectl set-hostname dns.ake.com
```
## show ip
```sh
ifconfig
```
## install bind9
```sh
sudo apt-get install bind9 bind9utils
```