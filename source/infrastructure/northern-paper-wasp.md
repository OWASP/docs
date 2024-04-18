# Server Configuration: northern-paper-wasp

This server hosts our Nextcloud instance, as well as various small, archived websites and Apache-based redirects.

## Initial Setup

We are using a cloud instance running Ubuntu 22.04 (current LTS as of deployment). Connect to the `root` account over SSH and update.

```bash
apt update
apt full-upgrade
```

Reboot if necessary.

Next we define the name of our host.

```bash
echo "northernpaperwasp" > /etc/hostname
hostname -F /etc/hostname
vim /etc/hosts
```

In that file, add the following lines below the first line, substituting the actual IPv4 and IPv6 of the instance in place of the NNN.NNN values:

```
NNN.NNN.NNN.NNN northernpaperwasp
NNNN:NNNN::NNNN:NNNN:NNNN:NNNN northernpaperwasp
```

Now we set the time zone.

```bash
dpkg-reconfigure tzdata
```

Use the arrow keys, tab, and spacebar set the timezone. We'll be using `US Eastern` for this instance.

### Defining a Non-Root User

We'll use the root account to set up the regular (but sudo-capable) user account.

```bash
adduser owasp
```

Define the password for the new user, and other information if desired. Then we add the user to the `sudo` group.

```bash
usermod -aG sudo owasp
groups owasp
```

The second command lists all the groups `owasp` is in. Ensure that includes `sudo`.

### SSH

We will backup the SSH configuration, and then adjust a few SSH settings.

```bash
cp /etc/ssh/sshd_config{,.bak}
vim /etc/ssh/sshd_config
```

Edit the file so the following lines have the given settings. These are
temporary, to enable us to add the SSH keys to the new account.

```
PasswordAuthentication yes
KbdInteractiveAuthentication yes
```

Now we restart the SSH service.

```bash
systemctl restart ssh
```

On the **remote machine** (the computer you’re connecting from), run the following command, where `NNN.NNN.NNN.NNN` is the IP address of the instance
you're connecting to:

```bash
ssh-copy-id owasp@NNN.NNN.NNN.NNN
```

When prompted, provide the password for the `owasp` account. Confirm your
ability to log in with `ssh owasp@NNN.NNN.NNN.NNN`.

On the host once again (as `owasp`), adjust the SSH configuration:

```bash
sudo vim /etc/ssh/sshd_config
```

Edit the file so the following lines have the given settings. (You may need
to add the `DebianBanner` line).

```
PermitRootLogin no
DebianBanner no
PasswordAuthentication no
KbdInteractiveAuthentication no
AuthorizedKeysFile      .ssh/authorized_keys
```

Restart the SSH service again.

```bash
sudo systemctl restart ssh
```

## Apache2

We will use Apache2 as our web server.

```bash
sudo apt install apache2
sudo vim /etc/apache2/apache2.conf
```

Edit the configuration file as following, adding or editing these:

```apache
ServerName 127.0.0.1
```

Scroll down to the section with all the directories, and add this entry. Be mindful to use **tabs**, not spaces, to be consistent with the rest of the file.

```apache
<DirectoryMatch /\.git/>
    Options FollowSymLinks
    AllowOverride None
    Require all denied
</DirectoryMatch>
```

Enable the event module and restart Apache2.

```bash
sudo a2enmod mpm_event
sudo systemctl restart apache2
```

Add the `owasp` user to the `www-data` group, which will be helpful for permissions.

```bash
sudo usermod -aG www-data owasp
```

Browse to the web server using the IP and ensure the Apache2 default page is appearing.

## PostgreSQL

We will use PostgreSQL as our database.

```bash
sudo apt install postgresql-14 postgresql-contrib
```

## Server Hardening

Let's improve our security before continuing.

### Firewall

We will enable the UFW firewall and allow only SSH and HTTP through.

```bash
sudo ufw allow 22
sudo ufw allow 80
sudo ufw allow 443
sudo ufw enable
```

### Secure Shared Memory

```bash
sudo vim /etc/fstab
```

At the bottom of the file, add these lines:

```
# Secure shared memory
tmpfs /run/shm tmpfs defaults,noexec,nosuid 0 0
```

The changes will take effect on next reboot.

### Lock Down `sudo` Privilege

Limit `sudo` privileges to only users in the `admin` group.

```bash
sudo groupadd admin
sudo usermod -aG admin owasp
sudo dpkg-statoverride --update --add root admin 4750 /bin/su
```

### Harden Network with `sysctl` Settings

```bash
sudo vim /etc/sysctl.conf
```

Edit the file,. uncommenting or adding the following lines:

```
# IP Spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Ignore ICMP broadcast requests
net.ipv4.icmp_echo_ignore_broadcasts = 1

# Disable source packet routing
net.ipv4.conf.all.accept_source_route = 0
net.ipv6.conf.all.accept_source_route = 0
net.ipv4.conf.default.accept_source_route = 0
net.ipv6.conf.default.accept_source_route = 0

# Ignore send redirects
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Block SYN attacks
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 2048
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 5

# Log Martians
net.ipv4.conf.all.log_martians = 1
net.ipv4.icmp_ignore_bogus_error_responses = 1

# Ignore ICMP redirects
net.ipv4.conf.all.accept_redirects = 0
net.ipv6.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
net.ipv6.conf.default.accept_redirects = 0

# Ignore Directed pings
net.ipv4.icmp_echo_ignore_all = 1
```

Reload `sysctl`. If there are any errors, fix the associated lines.

```bash
sudo sysctl -p
```

### Harden Apache2

Edit the Apache2 security configuration file:

```bash
sudo vim /etc/apache2/conf-available/security.conf
```

Change, uncomment, or add the following lines:

```apache
ServerTokens Prod
ServerSignature Off
TraceEnable Off
FileETag None
Header set X-Frame-Options: "sameorigin"
```

Enable the `headers` mod and restart Apache2.

```bash
sudo a2enmod headers
sudo systemctl restart apache2
```

### Setup ModSecurity

Install an up-to-date version of OWASP ModSecurity2 from [DigitalWave](https://modsecurity.digitalwave.hu/).

```bash
sudo apt-get -y install apt-transport-https lsb-release ca-certificates curl
sudo wget -qO - https://modsecurity.digitalwave.hu/archive.key | sudo apt-key add -
sudo sh -c 'echo "deb http://modsecurity.digitalwave.hu/ubuntu/ $(lsb_release -sc) main" > /etc/apt/sources.list.d/dwmodsec.list'
sudo sh -c 'echo "deb http://modsecurity.digitalwave.hu/ubuntu/ $(lsb_release -sc)-backports main" >> /etc/apt/sources.list.d/dwmodsec.list'
cat << EOF | sudo tee -a /etc/apt/preferences.d/99modsecurity
Package: *libnginx-mod-http-modsecurity*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900

Package: *libapache2-mod-security2*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900

Package: *modsecurity-crs*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900

Package: *libmodsecurity*
Pin: origin modsecurity.digitalwave.hu
Pin-Priority: 900
EOF
sudo apt update
sudo apt install libapache2-mod-security2
sudo apt remove modsecurity-crs
```

We copy the default configuration into place and edit it.

```bash
sudo cp /etc/modsecurity/modsecurity.conf{-recommended,}
sudo vim /etc/modsecurity/modsecurity.conf
```

Find and replace the following line:

```
SecRuleEngine On
```

Now we download and verify the latest OWASP CoreRuleSet according to the [documentation](https://coreruleset.org/docs/deployment/install/).

```bash
wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v4.0.0.tar.gz
wget https://github.com/coreruleset/coreruleset/releases/download/v4.0.0/coreruleset-4.0.0.tar.gz.asc
gpg --keyserver pgp.mit.edu --recv 0x38EEACA1AB8A6E72
gpg --verify coreruleset-4.0.0.tar.gz.asc v4.0.0.tar.gz
```

The output should indicate that the signature is good. If so, proceed to extract and install the CRS.

```bash
sudo mkdir /etc/crs4
sudo tar -xzvf v4.0.0.tar.gz --strip-components 1 -C /etc/crs4
sudo cp /etc/crs4/crs-setup.conf{.example,}
sudo vim /etc/apache2/apache2.conf
```

Add the following lines to the bottom of that file.

```apache2
IncludeOptional /etc/crs4/crs-setup.conf
IncludeOptional /etc/crs4/plugins/*-config.conf
IncludeOptional /etc/crs4/plugins/*-before.conf
IncludeOptional /etc/crs4/rules/*.conf
IncludeOptional /etc/crs4/plugins/*-after.conf
```

We also need to add the [CRS4 Nextcloud rule exclusion plugin](https://github.com/coreruleset/nextcloud-rule-exclusions-plugin).

```bash
cd /etc/crs4/plugins
sudo wget https://github.com/coreruleset/nextcloud-rule-exclusions-plugin/blob/main/plugins/nextcloud-rule-exclusions-before.conf
sudo wget https://github.com/coreruleset/nextcloud-rule-exclusions-plugin/blob/main/plugins/nextcloud-rule-exclusions-config.conf
```
