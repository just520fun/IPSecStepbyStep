# IPSecStepbyStep

Background
Raspberry Pi 3B+/OrangePi 4B

Software installation
Install strongswan IPsec server : 

sudo apt-get install strongswan libcharon-extra-plugins libstrongswan-extra-plugins

-------------
Step1 IKEv1

Hardware and Network Topology
Raspberry Pi - LAN (openwrt router) WLAN - Android Smart Phone(Samsung Note20 Ultra 5G)

Software Configuration

Configuration
IPSec

Backup the original IpSec configuration file:

sudo cp /etc/ipsec.conf /etc/ipsec.conf.bak

And edit it :

sudo vi /etc/ipsec.conf

As follow :

# ipsec.conf - strongSwan IPsec configuration file
config setup

conn %default
        keyexchange=ike

conn IPsec-Xauth-PSK
        keyexchange=ikev1
        authby=xauthpsk
        xauth=server
        ike=aes256-sha1-modp1024!
        left=192.168.1.114
        leftsubnet=0.0.0.0/0
        leftfirewall=yes
        right=%any
        rightsubnet=10.0.0.0/24
        rightsourceip=10.0.0.2/24
        auto=add

include /var/lib/strongswan/ipsec.conf.inc

Change 192.168.1.114 by your Pi address !
Secrets

Edit the secret files :

sudo nano /etc/ipsec.secrets

As follow :

# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.

# this file is managed with debconf and will contain the automatically created private key
include /var/lib/strongswan/ipsec.secrets.inc

192.168.1.114 : PSK "MyPresharedKey"

MyUser : XAUTH "MyPassword"

The IP should match the local IP of your Pi and you need to change the PSK, user(s) and password(s).

Then we can restart the service :

 sudo service ipsec restart
 or
 systemctl restart ipsec
 (systemctl restart strongswan  #in OrangePi)

And check the logs :

tail -n 100 /var/log/syslog 
or
systemctl status ipsec.service
(systemctl status strongswan  #in OrangePi)

The end should looks like this :

raspberrypi systemd[1]: Starting strongSwan IPsec IKEv1/IKEv2 daemon using ipsec.conf...
raspberrypi systemd[1]: Started strongSwan IPsec IKEv1/IKEv2 daemon using ipsec.conf.
raspberrypi ipsec[7373]: Starting strongSwan 5.2.1 IPsec [starter]...
raspberrypi charon: 00[DMN] Starting IKE charon daemon (strongSwan 5.2.1, Linux 4.9.35+, armv6l)
raspberrypi charon: 00[CFG] HA config misses local/remote address
raspberrypi charon: 00[LIB] plugin 'ha': failed to load - ha_plugin_create returned NULL
raspberrypi charon: 00[CFG] loading ca certificates from '/etc/ipsec.d/cacerts'
raspberrypi charon: 00[CFG] loading aa certificates from '/etc/ipsec.d/aacerts'
raspberrypi charon: 00[CFG] loading ocsp signer certificates from '/etc/ipsec.d/ocspcerts'
raspberrypi charon: 00[CFG] loading attribute certificates from '/etc/ipsec.d/acerts'
raspberrypi charon: 00[CFG] loading crls from '/etc/ipsec.d/crls'
raspberrypi charon: 00[CFG] loading secrets from '/etc/ipsec.secrets'
raspberrypi charon: 00[CFG]   loaded IKE secret for 192.168.1.4
raspberrypi charon: 00[CFG]   loaded EAP secret for MyUser
raspberrypi charon: 00[CFG] loaded 0 RADIUS server configurations
raspberrypi charon: 00[LIB] loaded plugins: charon test-vectors ldap pkcs11 aes rc2 sha1 sha2 md5 random nonce x509 revocation constraints pubkey pkcs1 pkcs7 pkcs8 pkcs12 pgp dnskey sshkey pem openssl gcrypt af-alg fips-prf gmp agent xcbc cmac hmac ctr ccm gcm curl attr kernel-netlink resolve socket-default farp stroke updown eap-identity eap-aka eap-md5 eap-gtc eap-mschapv2 eap-radius eap-tls eap-ttls eap-tnc xauth-generic xauth-eap xauth-pam tnc-tnccs dhcp lookip error-notify certexpire led addrblock unity
raspberrypi charon: 00[LIB] unable to load 5 plugin features (5 due to unmet dependencies)
raspberrypi charon: 00[LIB] dropped capabilities, running as uid 0, gid 0
raspberrypi charon: 00[JOB] spawning 16 worker threads
raspberrypi charon: 05[CFG] received stroke: add connection 'IPsec-Xauth-PSK'
raspberrypi charon: 05[CFG] adding virtual IP address pool 10.0.0.2/24
raspberrypi charon: 05[CFG] added configuration 'IPsec-Xauth-PSK'
raspberrypi ipsec[7373]: charon (7390) started after 1700 ms

----------------
Step2 IKEv1
Hardware and Network Topology
Raspberry Pi - LAN (openwrt router) LAN - OrangePi

Software Configuration

Raspberry Pi
/etc/ipsec.conf
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
        # strictcrlpolicy=yes
        # uniqueids = no

# Add connections here.
conn %default%
        ikelifetime=28800
        keylife=20m
        rekeymargin=3m
        keyingtries=1
        keyexchange=ikev1


conn test_ipsec
        left=192.168.1.107 #本端IP
        right=192.168.1.114 #对端IP
        leftsubnet=10.10.12.0/24 #本端子网
        leftauth=secret
        rightauth=secret
        rightsubnet=172.11.11.0/24 #对端子网
        auto=start
        leftid=192.168.1.107
        rightid=192.168.1.114
        ike=aes256-sha1-modp2048!
        esp=aes256-sha1!

/etc/ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.
192.168.1.107 192.168.1.114 : PSK 123456abcd

/etc/strongswan.conf
# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
        load_modular = yes
        duplicheck{
                enable = no
        }
        compress = yes
        plugins {
                include strongswan.d/charon/*.conf
        }
        dns1 = 114.114.114.114
        nbns1 = 114.114.114.114
}

include strongswan.d/*.conf

OrangePi
/etc/ipsec.conf
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
        # strictcrlpolicy=yes
        # uniqueids = no

# Add connections here.
conn %default%
        ikelifetime=28800
        keylife=20m
        rekeymargin=3m
        keyingtries=1
        keyexchange=ikev1


conn test_ipsec
#       aggressive=yes #野蛮模式
        left=192.168.1.114 #本端IP
        right=192.168.1.107 #对端IP
        leftsubnet=172.11.11.0/24 #本端子网
        leftauth=secret
        rightauth=secret
        rightsubnet=10.10.12.0/24 #对端子网
        auto=start
        leftid=192.168.1.114
        rightid=192.168.1.107
        ike=aes256-sha1-modp2048!
        esp=aes256-sha1!


/etc/ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.
192.168.1.107 192.168.1.114 : PSK 123456abcd

/etc/strongswan.conf
# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
        load_modular = yes
        duplicheck{
                enable = no
        }
        compress = yes
        plugins {
                include strongswan.d/charon/*.conf
        }
        dns1 = 114.114.114.114
        nbns1 = 114.114.114.114
}

include strongswan.d/*.conf

restart logs
Apr 21 10:27:21 raspberrypi ipsec[3062]: Starting strongSwan 5.7.2 IPsec [starter]...
Apr 21 10:27:21 raspberrypi charon: 00[CFG] loading ca certificates from '/etc/ipsec.d/cacerts'
Apr 21 10:27:21 raspberrypi charon: 00[CFG] loading aa certificates from '/etc/ipsec.d/aacerts'
Apr 21 10:27:21 raspberrypi charon: 00[CFG] loading ocsp signer certificates from '/etc/ipsec.d/ocspcerts'
Apr 21 10:27:21 raspberrypi charon: 00[CFG] loading attribute certificates from '/etc/ipsec.d/acerts'
Apr 21 10:27:21 raspberrypi charon: 00[CFG] loading crls from '/etc/ipsec.d/crls'
Apr 21 10:27:21 raspberrypi charon: 00[CFG] loading secrets from '/etc/ipsec.secrets'
Apr 21 10:27:21 raspberrypi charon: 00[CFG] expanding file expression '/var/lib/strongswan/ipsec.secrets.inc' failed
Apr 21 10:27:22 raspberrypi ipsec[3062]: charon (3076) started after 760 ms
Apr 21 10:27:22 raspberrypi charon: 04[CFG] received stroke: add connection 'test_ipsec'
Apr 21 10:27:22 raspberrypi charon: 04[CFG] added configuration 'test_ipsec'
Apr 21 10:27:22 raspberrypi charon: 07[CFG] received stroke: initiate 'test_ipsec'
Apr 21 10:27:22 raspberrypi charon: 07[IKE] initiating Main Mode IKE_SA test_ipsec[1] to 192.168.1.107
Apr 21 10:27:22 raspberrypi charon: 10[IKE] IKE_SA test_ipsec[1] established between 192.168.1.114[192.168.1.114]...192.168.1.107[192.168.1.107]
Apr 21 10:27:22 raspberrypi charon: 11[IKE] CHILD_SA test_ipsec{1} established with SPIs c65f000c_i c72cddaf_o and TS 172.11.11.0/24 === 10.10.12.0/24

-----------------------
Step3 IKEv2
Hardware and Network Topology
Raspberry Pi - LAN (openwrt router) LAN - OrangePi

Software Configuration

Raspberry Pi
/etc/ipsec.conf
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
        # strictcrlpolicy=yes
        # uniqueids = no

# Add connections here.
conn %default%
        ikelifetime=28800
        keylife=20m
        rekeymargin=3m
        keyingtries=1
        keyexchange=ikev2


conn test_ipsec
        left=192.168.1.107 #本端IP
        right=192.168.1.114 #对端IP
        leftsubnet=10.10.12.0/24 #本端子网
        leftauth=secret
        rightauth=secret
        rightsubnet=172.11.11.0/24 #对端子网
        auto=start
        leftid=192.168.1.107
        rightid=192.168.1.114
        ike=aes256-sha1-modp2048!
        esp=aes256-sha1!

/etc/ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.
192.168.1.107 192.168.1.114 : PSK 123456abcd

/etc/strongswan.conf
# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
        load_modular = yes
        duplicheck{
                enable = no
        }
        compress = yes
        plugins {
                include strongswan.d/charon/*.conf
        }
        dns1 = 114.114.114.114
        nbns1 = 114.114.114.114
}

include strongswan.d/*.conf

OrangePi
/etc/ipsec.conf
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
        # strictcrlpolicy=yes
        # uniqueids = no

# Add connections here.
conn %default%
        ikelifetime=28800
        keylife=20m
        rekeymargin=3m
        keyingtries=1
        keyexchange=ikev2


conn test_ipsec
        left=192.168.1.114 #本端IP
        right=192.168.1.107 #对端IP
        leftsubnet=172.11.11.0/24 #本端子网
        leftauth=secret
        rightauth=secret
        rightsubnet=10.10.12.0/24 #对端子网
        auto=start
        leftid=192.168.1.114
        rightid=192.168.1.107
        ike=aes256-sha1-modp2048!
        esp=aes256-sha1!


/etc/ipsec.secrets
# This file holds shared secrets or RSA private keys for authentication.

# RSA private key for this host, authenticating it to any other host
# which knows the public part.
192.168.1.107 192.168.1.114 : PSK 123456abcd

/etc/strongswan.conf
# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
        load_modular = yes
        duplicheck{
                enable = no
        }
        compress = yes
        plugins {
                include strongswan.d/charon/*.conf
        }
        dns1 = 114.114.114.114
        nbns1 = 114.114.114.114
}

include strongswan.d/*.conf

restart logs
Apr 21 10:47:28 raspberrypi systemd[1]: Stopped strongSwan IPsec IKEv1/IKEv2 daemon using ipsec.conf.
Apr 21 10:47:28 raspberrypi systemd[1]: Started strongSwan IPsec IKEv1/IKEv2 daemon using ipsec.conf.
pi@raspberrypi:~ $ Apr 21 10:47:28 raspberrypi ipsec[3594]: Starting strongSwan 5.7.2 IPsec [starter]...
Apr 21 10:47:29 raspberrypi charon: 00[CFG] loading ca certificates from '/etc/ipsec.d/cacerts'
Apr 21 10:47:29 raspberrypi charon: 00[CFG] loading aa certificates from '/etc/ipsec.d/aacerts'
Apr 21 10:47:29 raspberrypi charon: 00[CFG] loading ocsp signer certificates from '/etc/ipsec.d/ocspcerts'
Apr 21 10:47:29 raspberrypi charon: 00[CFG] loading attribute certificates from '/etc/ipsec.d/acerts'
Apr 21 10:47:29 raspberrypi charon: 00[CFG] loading crls from '/etc/ipsec.d/crls'
Apr 21 10:47:29 raspberrypi charon: 00[CFG] loading secrets from '/etc/ipsec.secrets'
Apr 21 10:47:29 raspberrypi charon: 00[CFG] expanding file expression '/var/lib/strongswan/ipsec.secrets.inc' failed
Apr 21 10:47:29 raspberrypi ipsec[3594]: charon (3608) started after 740 ms
Apr 21 10:47:29 raspberrypi charon: 03[CFG] received stroke: add connection 'test_ipsec'
Apr 21 10:47:29 raspberrypi charon: 03[CFG] added configuration 'test_ipsec'
Apr 21 10:47:29 raspberrypi charon: 06[CFG] received stroke: initiate 'test_ipsec'
Apr 21 10:47:29 raspberrypi charon: 06[IKE] initiating IKE_SA test_ipsec[1] to 192.168.1.107
Apr 21 10:47:29 raspberrypi charon: 09[IKE] establishing CHILD_SA test_ipsec{1}
Apr 21 10:47:29 raspberrypi charon: 10[IKE] IKE_SA test_ipsec[1] established between 192.168.1.114[192.168.1.114]...192.168.1.107[192.168.1.107]

-------------------------
Step4 IKEv2
Hardware and Network Topology
Raspberry Pi - LAN (openwrt router) LAN - OrangePi

Software Configuration
Raspberry Pi
/etc/ipsec.conf

/etc/ipsec.secrets

OrangePi
/etc/ipsec.conf

/etc/ipsec.secrets

restart logs

-------------------------
Step5 IKEv2
Hardware and Network Topology
Raspberry Pi - LAN (openwrt router) LAN - OrangePi

Software Configuration
Raspberry Pi
/etc/ipsec.conf

/etc/ipsec.secrets

OrangePi
/etc/ipsec.conf

/etc/ipsec.secrets

restart logs
