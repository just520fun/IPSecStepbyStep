# IPSecStepbyStep

Background
Raspberry Pi 3B+

Software installation
Install strongswan IPsec server : 

sudo apt-get install strongswan libcharon-extra-plugins libstrongswan-extra-plugins


Step1 IKEv1

Hardware and Network Topology
Raspberry Pi - LAN (openwrt router) WLAN - Android Smart Phone

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

And check the logs :

tail -n 100 /var/log/syslog 
or
systemctl status ipsec.service

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
