# **How to setup and configure a Certificate Authority?**

## What Is a Certificate Authority (CA)?

*A certificate authority, also known as a certification authority, is a trusted organization that verifies websites (and other entities) so that you know who you’re communicating with online. Their objective is to make the internet a more secure place for organizations and users alike. This means that they play a pivotal role in digital security.*

## Why we need Private CA?

*Having Private Certificate Authority will enable you to configure, test, and run programs that require encrypted connections between a client and a server. With a private CA, you can issue certificates for users, servers, or individual programs and services within your infrastructure.*

## Normal how it works?

*To request an SSL certificate from a CA like Verisign, you send them a Certificate Signing Request (CSR), and they give you a certificate in return that they signed using their root certificate and private key. All browsers have a copy (or access a copy from the operating system) of Verisign’s root certificate, so the browser can verify that your certificate was signed by a trusted CA.*

## Why Self-signed certificate wont work?

*Self-signed certificates are inherently not trusted by your browser because a certificate itself doesn't form any trust, the trust comes from being signed by a Certificate Authorit. Your browser simply doesn't trust your self-signed certificate as if it were a root certificate.*

### In this article, we’ll learn how to set up a private Certificate Authority

# Step 1 - Install Easy-RSA

Login to the server as **non-root** user

```shell
sudo apt update
sudo apt install easy-rsa -y
```

# Step 2 - Create a directory for PKI

Exectue the below commands as non-root user. Make sure that you do not use sudo to run any of the following commands, since your normal user should manage and interact with the CA without elevated privileges.

```shell
mkdir ~/pki
```

`<If you are using Linux, BSD, or a unix-like OS, open a shell and cd to the easy-rsa subdirectory. If you installed OpenVPN from an RPM or DEB file, the easy-rsa directory can usually be found in /usr/share/doc/packages/openvpn or /usr/share/doc/openvpn(it’s best to copy this directory to another location such as /etc/openvpn, before any edits, so that future OpenVPN package upgrades won’t overwrite your modifications). If you installed from a .tar.gz file, the easy-rsa directory will be in the top level directory of the expanded source tree.>`

To avoid the unwanted modifications we will create a new directory called easy-rsa in your home folder. We’ll use this directory to create symbolic links pointing to the easy-rsa package files that we’ve installed in the previous step. These files are located in the /usr/share/easy-rsa folder on the CA Server.

```shell
ln -s /usr/share/easy-rsa/* ~/pki/
```

Restrict only owner access to the new PKI directory, so that others cannot edit it.

```shell
chmod 700 /home/prem/pki
```

Now its time to initialize the PKI.

```shell
cd ~/pki
./easyrsa init-pki
```

pki directory will now contains all the files that are needed to create a Certificate Authority.

Next and the final command (build-ca) will build the certificate authority (CA) certificate and key.


# Step 3 - Creating a Certificate Authority

Before executing *build-ca* you need to crate and populate a **vars** file with some default values. 

* KEY_COUNTRY, 
* KEY_PROVINCE, 
* KEY_CITY, KEY_ORG, and 
* KEY_EMAIL parameters. 

Don’t leave any of these parameters blank. Create this file under *~/pki* folder

```shell
cd ~/pki
vi vars
```

Add the values like below edit each highlighted value to reflect your own organization info.

set_var EASYRSA_REQ_COUNTRY    "NL"
set_var EASYRSA_REQ_PROVINCE   "Flevoland"
set_var EASYRSA_REQ_CITY       "Almere"
set_var EASYRSA_REQ_ORG        "DevOpsHub"
set_var EASYRSA_REQ_EMAIL      "admin@devopshub.com"
set_var EASYRSA_REQ_OU         "Community"
set_var EASYRSA_ALGO           "ec"
set_var EASYRSA_DIGEST         "sha512"

Once done save and exit.

Now its time to run *build-ca* cmd

goto, ./pki and run the cmd with *build-ca* arg. This will create the root public and private key pair for your Certificate Authority

```shell
cd ~/pki
./easyrsa build-ca
```

In the output, you’ll see some lines about the OpenSSL version and you will be prompted to enter a passphrase for your key pair. Be sure to choose a strong passphrase, and note it down somewhere safe. You will need to input the passphrase any time that you need to interact with your CA, for example to sign or revoke a certificate.

You will also be asked to confirm the Common Name (CN) for your CA. The CN is the name used to refer to this machine in the context of the Certificate Authority. You can enter any string of characters for the CA’s Common Name but for simplicity’s sake, press ENTER to accept the default name.

To avoid prompted for a password every time you interact with your CA,

Run build-ca with nopass option

```shell
./easyrsa build-ca nopass
```

You now have two important files — ~/easy-rsa/pki/ca.crt and ~/easy-rsa/pki/private/ca.key — which make up the public and private components of a Certificate Authority.

* ca.crt is the CA’s public certificate file. Users, servers, and clients will use this certificate to verify that they are part of the same web of trust. Every user and server that uses your CA will need to have a copy of this file. All parties will rely on the public certificate to ensure that someone is not impersonating a system and performing a *Man-in-the-middle attack*.

* ca.key is the private key that the CA uses to sign certificates for servers and clients. If an attacker gains access to your CA and, in turn, your ca.key file, you will need to destroy your CA. This is why your ca.key file should only be on your CA machine and that, ideally, your CA machine should be kept offline when not signing certificate requests as an extra security measure.
With that, your CA is in place and it is ready to be used to sign certificate requests, and to revoke certificates.

