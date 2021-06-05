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
## Install bind9
```sh
sudo apt-get install bind9 bind9utils
```

[set up hyper-v on windows10]: <https://github.com/EknarongAphiphutthikul/Hyper-V>