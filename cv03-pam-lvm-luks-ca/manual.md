# Cviceni 3

## Vynuceni silnych hesel

### Knihovna libpam-pwquality

```bash
$ apt-get install libpam-pwquality
```

### Nastaveni

`/etc/pam.d/common-password`:

### vychozi nastaveni

```
password    requisite     pam_pwquality.so try_first_pass local_users_only retry=3 authtok_type=
```

### vynuceni sily hesla

```
password    requisite     pam_pwquality.so minlen=19 lcredit=0 ucredit=1 dcredit=1 ocredit=2
```

### historie hesel

```
password    required      pam_pwhistory.so remember=400 use_authtok
```

### zablokovani uctu

```
auth       required     pam_tally2.so deny=3 unlock_time=1800 even_deny_root
```

### slovnikova hesla

```
password required pam_cracklib.so retry=3 minlen=6 difok=3
```

### manual

```bash
$ man 8 pam_pwquality
```

## Nastaveni zdroje "jmenych" sluzeb

!Neplest s DNS!

```
/etc/nsswitch.conf
```

## Sifrovany souborovy system

### LVM2

```bash
$ pvcreate /dev/sdb
$ vgcreate vgbsa /dev/sdb
$ lvcreate -L 1G -n test vgbsa
```

### cryptsetup

```bash
$ cryptsetup -y -v luksFormat /dev/vgbsa/test
$ cryptsetup luksOpen /dev/vgbsa/test crypted
$ mkfs.ext4 /dev/mapper/crypted
$ mount /dev/mapper/crypted /mnt
```

### Persistetni pouziti

```bash
/etc/cryptotab
/etc/fstab
```

### Prace s klici

```bash
$ cryptsetup luksDump /dev/vgbsa/test
$ cryptsetup luksAddKey /dev/vgbsa/test /some/key/file
$ cryptsetup luksAddKey /dev/vgbsa/test /some/key/file -b /some/existing/key/file
$ cryptsetup luksAddKey /dev/vgbsa/test -S 6
$ cryptsetup luksRemoveKey /dev/vgbsa/test
$ cryptsetup luksKillSlot /dev/vgbsa/test 6
```

### Zaloha a obnoveni

```bash
$ cryptsetup luksHeaderBackup /dev/vgbsa/test --header-backup-file /mnt/vgbsa_test.img
$ cryptsetup luksHeaderRestore /dev/vgbsa/test --header-backup-file /mnt/vgbsa_test.img
```

## CA jednoduse (Easy RSA - 2.0)

(kopie easy-rsa)

```bash
$ cp -r /usr/share/easy-rsa/ /etc/CA2
```

### nastaveni CA a defaultni hodnoty pro certifikaty

```bash
$ vim /etc/CA2/vars
$ vim /etc/CA2/openssl-1.0.0.cnf
```

### vytvoreni CA

```bash
cd /etc/CA2
./clean-all
./build-ca
```

### vytvoreni certifikatu

```bash 
cd /etc/CA2
./build-key-server server.bsa-jindra.bsa
./build-key client.bsa-jindra.bsa
./build-key-pass client.bsa-jindra.bsa
```

### revokace certifikatu

```bash
cd /etc/CA2
./revoke-full
./list-crl
```

## EasyRSA 3.0

```
# Setup
$ cp -r /usr/share/easy-rsa/ /etc/CA3
$ cd /etc/CA3
$ mv vars.example vars
$ vim vars
$ ./easyrsa init-pki
$ ./easyrsa build-ca

# Server certificate 
$ ./easyrsa gen-req server.jindra.bsa
$ ./easyrsa sign server server.jindra.bsa

# Revoke
$ ./easyrsa revoke server.jindra.bas
$ ./easyrsa gen-crl
```

https://easy-rsa.readthedocs.io/en/latest/advanced/

## Lets encrypt certifikat

````
cd /opt && git clone https://github.com/lukas2511/dehydrated && cd dehydrated
cp ./docs/examples/hook.sh /etc/dehydrated/hook.sh && chmod +x /etc/dehydrated/hook.sh
vim /etc/dehydrated/config
```

konfigurace

```
DOMAINS_TXT=/etc/dehydrated/domains
WELLKNOWN="/var/www/dehydrated/.well-known/acme-challenge/"
CERTDIR="/etc/letsencrypt/live/"
CONTACT_EMAIL=skupaj@students.zcu.cz
HOOK=/etc/dehydrated/hook.sh
```

vytvorte registraci

```
dehydrated --register --accept-terms
```

Apache config

```
	Alias /.well-known/acme-challenge/ /var/www/dehydrated/.well-known/acme-challenge/
```

Nginx config

```
        location /.well-known/acme-challenge {
            alias   /var/www/dehydrated/.well-known/acme-challenge;
        }
# redirect
        if ($scheme = http) {
                return 302 https://$server_name$request_uri;
        }

```

## Step CA

Alternativne: https://github.com/smallstep/certificates/blob/master/docs/GETTING_STARTED.md
