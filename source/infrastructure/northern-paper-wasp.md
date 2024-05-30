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

Next, we’ll change the settings for the mpm_prefork module, which is needed for PHP 8.1 later, a dependency of Nextcloud.

```bash
sudo vim /etc/apache2/mods-available/mpm_prefork.conf
```

Set the file to the following...

```bash
<IfModule mpm_prefork_module>
        StartServers            2
        MinSpareServers         5
        MaxSpareServers         10
        MaxRequestWorkers       39
        MaxConnectionsPerChild  3000
</IfModule>
```

Save and close. Now we’ll enable the prefork module and restart Apache2.

```bash
sudo a2dismod mpm_event
sudo a2enmod mpm_prefork
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
sudo wget https://raw.githubusercontent.com/coreruleset/nextcloud-rule-exclusions-plugin/main/plugins/nextcloud-rule-exclusions-before.conf
sudo wget https://raw.githubusercontent.com/coreruleset/nextcloud-rule-exclusions-plugin/main/plugins/nextcloud-rule-exclusions-config.conf
```

### Setup Fail2Ban

Fail2Ban locks out IP addresses that repeatedly attempt invalid or malicious actions.

```bash
sudo apt install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo vim /etc/fail2ban/jail.local
```

To turn on various “jails”, scroll down to the # JAILS section. Place enabled = true under each jail name you want turned on. This is the list of jails we enabled for this server:

* sshd
* apache-auth
* apache-badbots
* apache-noscript
* apache-overflows
* apache-nohome
* apache-botsearch
* apache-fakegooglebot
* apache-modsecurity
* apache-shellshock
* recidive

Also add the `sshd-ddos` jail by including this entry at the bottom of the file:

```
[sshd-ddos]
mode = ddos
enabled = true
port = ssh
logpath = %(sshd_log)s
filter = sshd
```

Be sure you look through the jails and enable any additional jails that will be appropriate to your server’s configuration and applications.

For the `[recidive]` jail to work correctly, a couple of settings need to be changed in Fail2Ban’s configuration:

```bash
sudo cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local
sudo vim /etc/fail2ban/fail2ban.local
```

Change the following values:

```
# NEVER SET TO DEBUG!!! [recidive] jail is enabled
loglevel = INFO

dbpurgeage = 648000
```

Run the following command to ensure there are no errors:

```bash
sudo systemctl stop fail2ban
sudo fail2ban-client -x start
```

Finally, restart the fail2ban process.

```bash
sudo systemctl restart fail2ban
```

### Setup PSAD

We will use PSAD for intrusion detection and blocking.

```bash
sudo apt install psad
sudo iptables -A INPUT -j LOG
sudo iptables -A FORWARD -j LOG
sudo ip6tables -A INPUT -j LOG
sudo ip6tables -A FORWARD -j LOG
sudo vim /etc/psad/psad.conf
```

Change the following values:

```
EMAIL_ADDRESS owasp@localhost;
HOSTNAME northernpaperwasp;
ALERTING_METHODS noemail;
EMAIL_THROTTLE 100;
ALERT_ALL N;
ENABLE_AUTO_IDS_EMAIL N;
EMABLE_DNS_LOOKUPS N;
ENABLE_WHOIS_LOOKUPS N;
```

Now reload:

```bash
sudo psad -R; sudo psad --sig-update; sudo psad -H; sudo psad --Status
```

(If you get a warning about not finding a pidfile, it appears you may safely ignore that.)

We also need to tweak Fail2Ban so that it doesn’t start up before psad does. Otherwise, psad won’t be able to log correctly.

```bash
sudo vim /lib/systemd/system/fail2ban.service
```

In that file, add `ufw.service` and `psad.service` to the `After=` directive, so it looks something like this:

```
After=network.target iptables.service firewalld.service ip6tables.service ipset.service nftables.service ufw.service psad.service
```

Reload the daemons for systemctl and restart fail2ban.

```bash
sudo systemctl daemon-reload
sudo systemctl restart fail2ban
```

Now we adjust the UFW settings.

```bash
sudo ufw logging high
sudo vim /etc/ufw/before.rules
```

Add the following lines before the final `COMMIT`:

```
-A INPUT -j LOG
-A FORWARD -j LOG
```

Repeat this with `before6.rules`. Then, restart ufw and reload PSAD.

```bash
sudo systemctl restart ufw
sudo psad --fw-analyze
```

Restart the server, and ensure PSAD isn’t sending any system emails complaining about the firewall configuration. (Check system email by running `mail`).

## Installing Cloudflare Certificates

Because we're using Cloudflare as a proxy, we need to use a Cloudflare origin
certificate. Go to Cloudflare and generate an Origin Certificate from SSL/TLS -> Origin Server.

```bash
sudo vim /etc/ssl/certs/owasp.org.pem
```

Paste the origin certificate from Cloudflare.

Now run...

```bash
sudo chmod 644 /etc/ssl/certs/owasp.org.pem
sudo vim /etc/cloudflare/owasp.org.key
```

Paste the private key here, and then run...

```bash
sudo chmod 640 /etc/ssl/certs/owasp.org.key
```

Enable the SSL module for Apache2.

```bash
sudo a2enmod ssl
```

Now you can point your site configuration to these two files, like this:

```apache
                SSLEngine on
                SSLCertificateFile      /etc/ssl/certs/owasp.org.pem
                SSLCertificateKeyFile /etc/ssl/private/owasp.org.key
```

## Let's Encrypt Certbot

We need to install the Let’s Encrypt Certbot for generating certificates.

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
sudo certbot register
```

Follow the instructions to register with Let’s Encrypt.

We’ll actually generate certificates in a later step.

[SOURCE: Certbot (Certbot)](https://certbot.eff.org/lets-encrypt/ubuntufocal-apache)

### Scheduling Auto-Renewal

Now we need to schedule the autorenewal task.

```bash
sudo crontab -e
```

Add the following line to the end:

```
41 5 * * * /usr/bin/certbot renew
```

This will run the renewal script once a day at 5:41am. (Let’s Encrypt asks that a random time be used by each user, to spread out server load.)

## Nextcloud

We need to install the dependencies for Nextcloud.

```bash
sudo apt install php8.1 php8.1-curl php8.1-gd php8.1-xml php8.1-mbstring php8.1-xml php8.1-zip php8.1-pgsql php8.1-bz2 php8.1-intl php8.1-imap php8.1-bcmath php8.1-gmp php8.1-apcu php8.1-memcached php8.1-redis php8.1-imagick ffmpeg
sudo phpenmod bcmath gmp
```

Now we can install Nextcloud itself, which we'll unpack from the community
archive.

```bash
cd /tmp
curl -LO https://download.nextcloud.com/server/releases/latest.tar.bz2
curl -LO https://download.nextcloud.com/server/releases/latest.tar.bz2.sha256
shasum -a 256 -c latest.tar.bz2.sha256 < latest.tar.bz2
```

Ensure the output of that last command is `latest.tar.bz2: OK`.

```bash
rm latest.tar.bz2.sha256
sudo tar -C /opt -xvjf /tmp/latest.tar.bz2
vim /tmp/nextcloud.sh
```

Paste the following:

```bash
ocpath='/opt/nextcloud'
htuser='www-data'
htgroup='www-data'
rootuser='root'

printf "Creating possible missing Directories\n"
mkdir -p $ocpath/data
mkdir -p $ocpath/assets
mkdir -p $ocpath/updater

printf "chmod Files and Directories\n"
find ${ocpath}/ -type f -print0 | xargs -0 chmod 0640
find ${ocpath}/ -type d -print0 | xargs -0 chmod 0750
chmod 755 ${ocpath}

printf "chown Directories\n"
chown -R ${rootuser}:${htgroup} ${ocpath}/
chown -R ${htuser}:${htgroup} ${ocpath}/apps/
chown -R ${htuser}:${htgroup} ${ocpath}/assets/
chown -R ${htuser}:${htgroup} ${ocpath}/config/
chown -R ${htuser}:${htgroup} ${ocpath}/data/
chown -R ${htuser}:${htgroup} ${ocpath}/themes/
chown -R ${htuser}:${htgroup} ${ocpath}/updater/

chmod +x ${ocpath}/occ

printf "chmod/chown .htaccess\n"
if [ -f ${ocpath}/.htaccess ]
then
chmod 0664 ${ocpath}/.htaccess
chown ${rootuser}:${htgroup} ${ocpath}/.htaccess
fi
if [ -f ${ocpath}/data/.htaccess ]
then
chmod 0664 ${ocpath}/data/.htaccess
chown ${rootuser}:${htgroup} ${ocpath}/data/.htaccess
fi
```

Run the file.

```bash
sudo bash /tmp/nextcloud.sh
```

That will create the directories.

### Domain and Certificates

We first set up the domain name and the certificates. For our configuration, we used `cloud.owasp.org`.

We generate a certificate like this:

```bash
sudo certbot certonly --apache -d cloud.owasp.org
```

In the output for the certbot command, take note of the paths where the certificate and chain were saved. You’ll need that in the next step.

### Apache2 Configuration

We create an Apache2 site configuration for Nextcloud.

```bash
sudo vim /etc/apache2/sites-available/cloud.conf
```

Set the contents to…

```apache
<IfModule mod_ssl.c>
    <VirtualHost *:443>
        ServerName cloud.mousepawmedia.com
        DocumentRoot /opt/nextcloud

        SSLEngine on
        SSLCertificateFile /etc/letsencrypt/live/cloud.owasp.org/fullchain.pem
        SSLCertificateKeyFile /etc/letsencrypt/live/cloud.owasp.org/privkey.pem

        Include /etc/letsencrypt/options-ssl-apache.conf
        Header always set Strict-Transport-Security "max-age=31536000"
        Header always set Content-Security-Policy upgrade-insecure-requests

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        <Directory /opt/nextcloud>
            Require all granted
            Options +FollowSymLinks
            AllowOverride All
            Satisfy Any

            <IfModule mod_dave.c>
                    Dav off
            </IfModule>

            SetEnv HOME /opt/nextcloud
            SetEnv HTTP_HOME /opt/nextcloud
        </Directory>

        BrowserMatch "MSIE [2-6]" \
            nokeepalive ssl-unclean-shutdown \
            downgrade-1.0 force-response-1.0
        # MSIE 7 and newer should be able to use keepalive
        BrowserMatch "MSIE [17-9]" ssl-unclean-shutdown
    </VirtualHost>
</IfModule>
```

Enable the site and enable Apache2.

```bash
sudo a2ensite cloud
sudo a2enmod headers php8.1
sudo systemctl restart apache2
```

If you go to your domain name in a web browser, you should see the Nextlcoud setup screen. We will need to do some additional steps before proceeding with that.

### Database

Connect to the PostgreSQL instance like this:

```bash
sudo su - postgres
psql
```

Set up the necessary users and databases, substituting a real value for `'password'`.

```sql
CREATE ROLE nextcloud WITH LOGIN PASSWORD 'password';
CREATE DATABASE nextcloud OWNER nextcloud;
\q
```

You can exit out of the `postgres` shell session too with `exit`.

Now configure PHP to work with PostgreSQL.

```
sudo phpenmod pgsql
sudo systemctl restart apache2
```

### Redis

We also install and configure Redis.

```bash
sudo apt install redis-server
sudo vim /etc/redis/redis.conf
```

Add or change the following lines, substituting a real password in place of `PASSWORDHERE`:

```
supervised systemd

bind 127.0.0.1 -::1

requirepass PASSWORDHERE
```

Run the following to start Redis:

```bash
sudo systemctl restart redis-server
redis-cli
```

In the interactive session that appears, enter the following commands at the prompt (>). Responses are displayed inline without the leading > below:

```
> auth PASSWORDHERE
OK
> ping
PONG
> quit
```

That confirms Redis is configured.

### Configuring Nextcloud

Go to the new Nextcloud site. On the setup screen, specify an admin account.

Click Storage and Database, set the Data folder to `/opt/nextcloud/data`. Select `PostgreSQL` for the database, and provide the database user, password, and database name. The fourth field should be `localhost:5432`.

Click `Finish setup`.

### PHP Configuration

Nextcloud recommends a few tweaks to php.ini.

```bash
sudo vim /etc/php/8.1/apache2/php.ini
```

Add or edit (or uncomment) the following lines:

```
date.timezone = America/New_York
memory_limit = 512M
post_max_size = 512M
upload_max_filesize = 512M

opcache.enable=1
opcache.enable_cli=1
opcache.memory_consumption=128
opcache.interned_strings_buffer=8
opcache.max_accelerated_files=10000
opcache.revalidate_freq=1
opcache.save_comments=1
opcache.jit = 1255
opcache.jit_buffer_size = 128M
```

Now modify the settings for APCu:

```bash
sudo vim /etc/php/8.1/mods-available/apcu.ini
```

Add the following line:

```
apc.enable_cli=1
```

Restart Apache2.

```bash
sudo systemctl restart apache2
```

### Configuring Memory Caching


To improve performance, we’ll enable memory caching. We are using APCu and Redis, so we need to enable this for Nextcloud.

```bash
sudo vim /opt/nextcloud/config/config.php
```

Add the following lines, substituting your actual Redis password in place of `PASSWORDHERE`, before the final `);`:

```php
  'filelocking.enabled' => true,
  'memcache.local' => '\OC\Memcache\APCu',
  'memcache.distributed' => '\OC\Memcache\Redis',
  'memcache.locking' => '\OC\Memcache\Redis',
  'redis' => array(
      'host' => 'localhost',
      'port' => 6379,
      'timeout' => 0.0,
      'password' => 'PASSWORDHERE'
  ),
```

Restart Apache2.

```bash
sudo systemctl restart apache2
```

### Set Up Cronjob

It is recommended to use Cron for background tasks.

```bash
sudo crontab -u www-data -e
```

Add the following line:

```
*/15  *  *  *  * php -f /opt/nextcloud/cron.php
```

Finally, in the Nextcloud Admin pane, go to `Basic Settings` and select the
`Cron` option.

[SOURCE: Background Jobs Configuration (Nextcloud)](https://docs.nextcloud.com/server/10/admin_manual/configuration_server/background_jobs_configuration.html)

### S3 Bucket Storage

We’ll be setting an S3 Object Storage Bucket as the primary data host.

```{note}
Do NOT enable the Encryption app; it’s not compatible with S3!
```

Create an S3-compatible bucket, and acquire the corresponding access key for full read/write permissions.

Then edit the Nextcloud configuration:

```bash
sudo vim /opt/nextcloud/config/config.php
```

Add the following entries to that file, before the closing `);`, substituting your values (especially in place of `BUCKETNAME`, `ACCESSKEY` and `SECRETKEY`).

```php
  'objectstore' => array(
      'class' => '\\OC\\Files\\ObjectStore\\S3',
      'arguments' => array(
        'bucket' => 'BUCKETNAME',
        'autocreate' => false,
        'key' => 'ACCESSKEY',
        'secret' => 'SECRETKEY',
        'hostname' => 'nyc3.digitaloceanspaces.com',
        'port' => 443,
        'use_ssl' => true,
        'region' => 'nyc3',
      ),
  ),
```

Restart Apache2. Nextcloud will start storing in that bucket instead of on the system disk.

[SOURCE: Deploy Nextcloud with Object Storage (Scaleway)](https://www.scaleway.com/en/docs/install-and-configure-nextcloud-object-storage/)

[SOURCE: Spaces API Reference (DigitalOcean)](https://docs.digitalocean.com/reference/api/spaces-api/)

### Pretty URLs

The default URLs for NextCloud all include index.php, which isn’t very nice to look at all the time. Let’s fix this.

```bash
sudo vim /opt/nextcloud/config/config.php
```

In that file, modify the following lines or add them somewhere before the final `);`:

```php
  'overwrite.cli.url' => 'https://cloud.owasp.org/',
  'htaccess.RewriteBase' => '/',

```

Then run:

```bash
sudo -u www-data php /opt/nextcloud/occ maintenance:update:htaccess
sudo systemctl restart apache2
```

### Changing Default Files

To change or remove the default files, change what is found in `/opt/nextcloud/core/skeleton`. For our instance of Nextcloud, we’ve removed all these files.
