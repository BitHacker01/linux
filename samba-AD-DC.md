
# How to Set Up Samba as an Active Directory Domain Controller on Ubuntu


## Step1: Set Hostname & Static IP for Sever identity

### 1.Set a proper hostname:

`sudo hostnamectl set-hostname dc1.abdu.local`

or

`sudo nano /etc/hosts`

`127.0.0.1       localhost
192.168.1.100    dc1.abdu.local	dc1`

### 2.Setting Static IP Address


`sudo nano /etc/netplan/01-network.yaml`

```
network:
  version: 2
  ethernets:
    wlo1:
      dhcp4: false

      addresses:
        - 192.168.1.100/24

      routes:
        - to:  default
          via: 192.168.1.1

      nameservers:
        addresses:
          - 127.0.0.1
          - 8.8.8.8

        search:
          -  abdu.local

```

`sudo netplan apply`

## Step 2: Install Samba



```
sudo apt update
sudo apt install -y samba samba-dsdb-modules samba-vfs-modules \
  krb5-user winbind libnss-winbind libpam-winbind \
  dnsutils acl
  ```
  or
  
  `sudo apt install samba-ad-dc krb5-user bind9-dnsutils`
  
  or
  
  `sudo apt install -y samba winbind krb5-config smbclient dnsutils net-tools`

## Step 3: Stop and Disable Conflicting Services

```
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.backup
sudo systemctl disable --now smbd nmbd winbind systemd-resolved.services
```
or
```
# Stop all existing Samba and related services
sudo systemctl stop samba-ad-dc smbd nmbd winbind 2>/dev/null || true
sudo systemctl disable samba-ad-dc smbd nmbd winbind 2>/dev/null || true

# Remove existing Samba configuration (back it up first)
sudo mv /etc/samba/smb.conf /etc/samba/smb.conf.backup

# Remove existing Samba database files
sudo find /var/lib/samba -name "*.tdb" -delete
sudo find /var/lib/samba -name "*.ldb" -delete
sudo rm -rf /var/lib/samba/private
```
## Step 4: Configure DNS

Samba's internal DNS needs to be the resolver for the domain. Update  `/etc/resolv.conf`:

`sudo nano /etc/resolv.conf`
```
nameserver 127.0.0.1
search abdu.local
```

or 

`ln -s /run/systemd/resolve/resolv.conf /etc/resolv.conf` 


## Step 5: Provision the Domain

`sudo samba-tool domain provision  --use-rfc2307  --interactive`

`realm: ABDU.LOCAL , dns forwarder: 8.8.8.8`
Enter until set password

or
```
sudo samba-tool domain provision \ --use-rfc2307 \ --realm=CORP.EXAMPLE.COM \ --domain=CORP \ --server-role=dc \ --dns-backend=SAMBA_INTERNAL \ --adminpass='AdminPassword123!'
```
## Step 6: Configure Kerberos
`sudo cp /var/lib/samba/private/krb5.conf /etc/krb5.conf`

## Step 7: Start the Samba AD DC Service

```
sudo systemctl unmask samba-ad-dc
sudo systemctl enable samba-ad-dc
sudo systemctl start samba-ad-dc
sudo systemctl status samba-ad-dc
```

## Step8: Test 
### 1.Samba-ad-dc is active or not 

`sudo netstat -antp | egrep 'smbd|samba'`

if not listen samba then 

`reboot` see again

### 2.set client  DNS as Server IP
`win+R -> ncpa.cpl -> ipv4 -> properties -> dns : 192.168.1.100 , alernative dns: 8.8.8.8`

`ping dc1`

### 3.join windows to Domain
`system -> about -> domain or workgroup -> change -> domain: abdu.local -> ok -> restart`

### 4.login as Administrator in the domain
`other -> username: ABDU\administrator -> password: **** -> login

## Step9: If  have any isuses to check it out the below links 

1. https://oneuptime.com/blog/post/2026-03-02-how-to-set-up-samba-as-an-active-directory-domain-controller-on-ubuntu/view

2. https://ubuntu.com/server/docs/how-to/samba/provision-samba-ad-controller/

3. https://youtu.be/tgBuvA6J-_8?si=qOVeKNTt8yQesdqB

