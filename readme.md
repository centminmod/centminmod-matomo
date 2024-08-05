# Centmin Mod Matomo Installation Guide

1. Prepare your intended domain name or subdomain's DNS and Matomo MariaDB MySQL database name, user and password and Maxmind geolocation database license registration. Below guide will use 

* subdomain hostname = `analytics.domain.com` 
* MariaDB database name = `matomodb`
* MariaDB database username = `matomo_user`
* MariaDB database username password = `matomopass`

a. Update `analytics.domain.com` DNS A record to point to Centmin Mod server's IPv4 IP address

b. Edit /etc/my.cnf and add `local-infile=1` to `[mysql]` and `[mysqld]` sections and remove existing `local-infile=0`

```
[mysql]
local-infile=1
max_allowed_packet = 64M

[mysqld]
local-infile=1
```

Then restart MariaDB MySQL server via command shortcut or systemctl command

```
mysqlrestart
```

or

```
systemctl restart mariadb
```

c. Create MariaDB database name and user/pass using Centmin Mod's addons/mysqladmin_shell.sh tool

```
/usr/local/src/centminmod/addons/mysqladmin_shell.sh createuserdb matomodb matomo_user matomopass
```

Or if you want a more secure password use included `pwgen` tool to generate. Be sure to write down the output from `echo $newpass`

```
newpass=$(pwgen -1cny 27 | tr -d '\!@#\$%^&*()_+[]{}|;:,.<>?\\`')
echo $newpass
burie1quaiz2OhcauDohwaich

/usr/local/src/centminmod/addons/mysqladmin_shell.sh createuserdb matomodb matomo_user $newpass
```

There is also a Node.js dependency for some Matomo Plugins i.e. Performance Audit Plugin. So install Node.js via Centmin Mod `addons/nodejs.sh` addon installer.

```
/usr/local/src/centminmod/addons/nodejs.sh install
```
```
node --version
v20.16.0

npm --version
10.8.1
```

And YUM dependencies for Performance Audit Plugin. On Centmin Mod LEMP stack with AlmaLinux 9:

List dependencies

```
yum -q list alsa-lib.x86_64 atk.x86_64 cups-libs.x86_64 gtk3.x86_64 ipa-gothic-fonts libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXrandr.x86_64 libXScrnSaver.x86_64 libXtst.x86_64 pango.x86_64 xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-fonts-cyrillic xorg-x11-fonts-misc xorg-x11-fonts-Type1 xorg-x11-utils | tr -s ' ' | column -t
Installed                       Packages             
alsa-lib.x86_64                 1.2.10-2.el9         @appstream
atk.x86_64                      2.36.0-5.el9         @appstream
cups-libs.x86_64                1:2.3.3op2-27.el9_4  @baseos
gtk3.x86_64                     3.24.31-2.el9        @appstream
libXcomposite.x86_64            0.4.5-7.el9          @appstream
libXcursor.x86_64               1.2.0-7.el9          @appstream
libXdamage.x86_64               1.1.5-7.el9          @appstream
libXext.x86_64                  1.3.4-8.el9          @appstream
libXi.x86_64                    1.7.10-8.el9         @appstream
libXrandr.x86_64                1.5.2-8.el9          @appstream
libXtst.x86_64                  1.2.3-16.el9         @appstream
pango.x86_64                    1.48.7-3.el9         @appstream
Available                       Packages             
libXScrnSaver.x86_64            1.2.3-10.el9         appstream
xorg-x11-fonts-100dpi.noarch    7.5-33.el9           appstream
xorg-x11-fonts-75dpi.noarch     7.5-33.el9           appstream
xorg-x11-fonts-Type1.noarch     7.5-33.el9           appstream
xorg-x11-fonts-cyrillic.noarch  7.5-33.el9           appstream
xorg-x11-fonts-misc.noarch      7.5-33.el9           appstream
xorg-x11-utils.x86_64           7.5-40.el9           appstream
```

Install dependencies

```
yum -y install alsa-lib.x86_64 atk.x86_64 cups-libs.x86_64 gtk3.x86_64 ipa-gothic-fonts libXcomposite.x86_64 libXcursor.x86_64 libXdamage.x86_64 libXext.x86_64 libXi.x86_64 libXrandr.x86_64 libXScrnSaver.x86_64 libXtst.x86_64 pango.x86_64 xorg-x11-fonts-100dpi xorg-x11-fonts-75dpi xorg-x11-fonts-cyrillic xorg-x11-fonts-misc xorg-x11-fonts-Type1 xorg-x11-utils --skip-broken
```

2. Ensure you have Letsencrypt integration and Nginx GeoIP 2 Lite Nginx module support enabled by setting up in Centmin Mod persistent config file `/etc/centminmod/custom_config.inc` the variables for:

```
LETSENCRYPT_DETECT='y'
NGINX_GEOIPTWOLITE='y'
NGXDYNAMIC_GEOIPTWOLITE='y'
```

You can run these 3 commands to automate this:

```
grep 'LETSENCRYPT_DETECT' /etc/centminmod/custom_config.inc || echo "LETSENCRYPT_DETECT='y'" >> /etc/centminmod/custom_config.inc
grep 'NGINX_GEOIPTWOLITE' /etc/centminmod/custom_config.inc || echo "NGINX_GEOIPTWOLITE='y'" >> /etc/centminmod/custom_config.inc
grep 'NGXDYNAMIC_GEOIPTWOLITE' /etc/centminmod/custom_config.inc || echo "NGXDYNAMIC_GEOIPTWOLITE='y'" >> /etc/centminmod/custom_config.inc
```

If you are using Cloudflare for, then also setup Cloudflare DNS API validation as per https://centminmod.com/letsencrypt-freessl.html#dns. Cloudflare API Token variables are set in persistent config file at `/etc/centminmod/custom_config.inc`. This method is recommended if your Centmin Mod Nginx domain is behind Cloudflare orange cloud enabled proxy and you have Cloudflare Full or Full Strict SSL mode enabled.

Cloudflare API Tokens, requires you to create your Cloudflare Token API with permissions for read access to Zone.Zone, and edit/write access to Zone.DNS, across all Zones at https://dash.cloudflare.com/profile/api-tokens and to grab your Cloudflare Account ID from any of your Cloudflare domain's main dashboard's right side column listing.

```
LETSENCRYPT_DETECT='y'
NGINX_GEOIPTWOLITE='y'
NGXDYNAMIC_GEOIPTWOLITE='y'
CF_DNSAPI_GLOBAL='y'
CF_Token="YOUR_CF_TOKEN"
CF_Account_ID="YOUR_CF_ACCOUNT_ID"
```

3. Update Nginx to 1.27.0 via centmin.sh menu option 4

```
--------------------------------------------------------
     Centmin Mod Menu 140.00beta01 centminmod.com     
--------------------------------------------------------
1).  Centmin Install
2).  Add Nginx vhost domain
3).  NSD setup domain name DNS
4).  Nginx Upgrade / Downgrade
5).  PHP Upgrade / Downgrade
6).  MySQL User Database Management
7).  Persistent Config File Management
8).  Option Being Revised (TBA)
9).  Option Being Revised (TBA)
10). Memcached Server Re-install
11). MariaDB MySQL Upgrade & Management
12). Zend OpCache Install/Re-install
13). Install/Reinstall Redis PHP Extension
14). SELinux disable
15). Install/Reinstall ImagicK PHP Extension
16). Change SSHD Port Number
17). Multi-thread compression: zstd,pigz,pbzip2,lbzip2
18). Suhosin PHP Extension install
19). Install FFMPEG and FFMPEG PHP Extension
20). NSD Install/Re-Install
21). Data Transfer
22). Add Wordpress Nginx vhost + Cache Plugin
23). Update Centmin Mod Code Base
24). Exit
--------------------------------------------------------
Enter option [ 1 - 24 ] 4
--------------------------------------------------------
```
```
Nginx Upgrade - Would you like to continue? [y/n] y

Current Nginx Version: 1.27.0 (260724-025453-almalinux8-kvm-561a713-br-a71f931)

Install which version of Nginx? (version i.e. type 1.27.0): 1.27.0

Do you still want to continue? [y/n] y
```

On Centmin Mod Nginx upgrade completion, you can verify if Nginx was build with GeoIP 2 Lite nginx module support using below command when installed woult return the value of `ngx_http_geoip2_module`

```
nginx -V 2>&1 | grep -o 'ngx_http_geoip2_module'
ngx_http_geoip2_module
```

```
nginx -V
nginx version: nginx/1.27.0 (040824-141522-almalinux8-kvm-da33abe-br-a71f931)
built by gcc 13.2.1 20231205 (Red Hat 13.2.1-6) (GCC) 
built with OpenSSL 1.1.1k  FIPS 25 Mar 2021
TLS SNI support enabled
```
> configure arguments: --with-ld-opt='-Wl,-E -L/usr/local/zlib-cf/lib -L/usr/local/nginx-dep/lib -ljemalloc -Wl,-z,relro,-z,now -Wl,-rpath,/usr/local/zlib-cf/lib:/usr/local/nginx-dep/lib -pie -flto=2 -flto-compression-level=1 -fuse-ld=gold' --with-cc-opt='-I/usr/local/zlib-cf/include -I/usr/local/nginx-dep/include -m64 -march=native -fPIC -g -O3 -fstack-protector-strong -flto=2 -flto-compression-level=1 -fuse-ld=gold --param=ssp-buffer-size=4 -Wformat -Wno-pointer-sign -Wimplicit-fallthrough=0 -Wno-cast-align -Wno-implicit-function-declaration -Wno-builtin-declaration-mismatch -Wno-deprecated-declarations -Wno-int-conversion -Wno-unused-result -Wno-vla-parameter -Wno-maybe-uninitialized -Wno-return-local-addr -Wno-array-parameter -Wno-alloc-size-larger-than -Wno-address -Wno-array-bounds -Wno-discarded-qualifiers -Wno-stringop-overread -Wno-stringop-truncation -Wno-missing-field-initializers -Wno-unused-variable -Wno-format -Wno-error=unused-result -Wno-missing-profile -Wno-stringop-overflow -Wno-free-nonheap-object -Wno-discarded-qualifiers -Wno-bad-function-cast -Wno-dangling-pointer -Wno-array-parameter -fcode-hoisting -Wno-cast-function-type -Wno-format-extra-args -Wp,-D_FORTIFY_SOURCE=2' --prefix=/usr/local/nginx --sbin-path=/usr/local/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --build=040824-141522-almalinux8-kvm-da33abe-br-a71f931 --with-compat --without-pcre2 --with-http_stub_status_module --with-http_secure_link_module --with-libatomic --with-http_gzip_static_module --add-dynamic-module=../ngx_brotli --add-module=../zstd-nginx-module --add-dynamic-module=../ngx_http_geoip2_module --with-http_sub_module --with-http_addition_module --with-http_image_filter_module=dynamic --with-http_geoip_module --with-stream_geoip_module --with-stream_realip_module --with-stream_ssl_preread_module --with-threads --with-stream --with-stream_ssl_module --with-http_realip_module --add-dynamic-module=../ngx-fancyindex-0.4.2 --add-module=../ngx_cache_purge-2.5.3 --add-dynamic-module=../ngx_devel_kit-0.3.2 --add-dynamic-module=../set-misc-nginx-module-0.33 --add-dynamic-module=../echo-nginx-module-0.63 --add-module=../redis2-nginx-module-0.15 --add-module=../ngx_http_redis-0.4.0-cmm --add-module=../memc-nginx-module-0.20 --add-module=../srcache-nginx-module-0.33 --add-dynamic-module=../headers-more-nginx-module-0.37 --with-pcre-jit --with-zlib=../zlib-cloudflare-1.3.3 --with-zlib-opt=-fPIC --with-http_ssl_module --with-http_v2_module

4. Create Centmin Mod Nginx Vhost with HTTPS Letsencrypt SSL certificate support using Centmin Mod `nv` command line tool (you can also use centmin.sh menu option 2 shell menu for interactive creation) as outlined at https://centminmod.com/nginx_domain_dns_setup.html:

`nv` command options

```
nv

Usage: /bin/nv [-d yourdomain.com] [-s y|n|yd|le|led|lelive|lelived] [-u ftpusername]

  -d  yourdomain.com or subdomain.yourdomain.com
  -s  ssl self-signed create = y or n or https only vhost = yd
  -s  le - letsencrypt test cert or led test cert with https default
  -s  lelive - letsencrypt live cert or lelived live cert with https default
  -u  your FTP username

  example:

  /bin/nv -d yourdomain.com -s y -u ftpusername
  /bin/nv -d yourdomain.com -s n -u ftpusername
  /bin/nv -d yourdomain.com -s yd -u ftpusername
  /bin/nv -d yourdomain.com -s le -u ftpusername
  /bin/nv -d yourdomain.com -s led -u ftpusername
  /bin/nv -d yourdomain.com -s lelive -u ftpusername
  /bin/nv -d yourdomain.com -s lelived -u ftpusername
```

For Centmin Mod Nginx vhost with Letsencrypt HTTPS default changing `ftpusername` to your desired pure-ftpd virtual FTP username. The password will be auto generated and displayed when creation run is completed.

```
nv -d analytics.domain.com -s lelived -u ftpusername
```

You'll get information for your Nginx vhost like including the pure-ftpd virtual FTP user credentials https://centminmod.com/ftp.html:

```
-------------------------------------------------------------
FTP hostname : xxx.xxx.xxx.xxx
FTP port : 21
FTP mode : FTP (explicit SSL)
FTP Passive (PASV) : ensure is checked/enabled
FTP username created for analytics.domain.com : ftpusername
FTP password created for analytics.domain.com : O2DxxxxxxJRHWKy
-------------------------------------------------------------
vhost for analytics.domain.com created successfully

AUDITD_ENABLE is set to = n

domain: http://analytics.domain.com
vhost conf file for analytics.domain.com created: /usr/local/nginx/conf/conf.d/analytics.domain.com.conf

vhost ssl for analytics.domain.com created successfully

domain: https://analytics.domain.com
vhost ssl conf file for analytics.domain.com created: /usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf

upload files to /home/nginx/domains/analytics.domain.com/public
vhost log files directory is /home/nginx/domains/analytics.domain.com/log
```

And also shows log location if you need to revisit the credential info or troubleshoot Nginx vhost creation

```
-------------------------------------------------------------
vhost for analytics.domain.com setup successfully
analytics.domain.com setup info log saved at: 
/root/centminlogs/centminmod_040824-143037_nginx_addvhost_nv.log
-------------------------------------------------------------
```

`nv` command and centmin.sh menu option 2 or 22 will also save a `*-remove-*.log` like `centminmod_040824-143037_nginx_addvhost_nv-remove-cmds-analytics.domain.com.log` with commands to remove the created Nginx vhost

```
ls -lAhrt /root/centminlogs/ | tail -2
-rw-r--r-- 1 root root 1.4K Aug  4 14:30 centminmod_040824-143037_nginx_addvhost_nv-remove-cmds-analytics.domain.com.log
-rw-r--r-- 1 root root 8.5K Aug  4 14:30 centminmod_040824-143037_nginx_addvhost_nv.log
```

You can optionally setup some SSH command aliases commands for easier navigation of your Nginx vhost. Type in SSH session:

```
alias myconf='nano /usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf'
alias mylog='pushd /home/nginx/domains/analytics.domain.com/log/'
alias mypub='pushd /home/nginx/domains/analytics.domain.com/public/'
```

and add these 3 lines into your `/root/.bashrc` file.

Then you'll be able to type:

* `myconf` to launch nano text editor to edit Nginx vhost config file
* `mylog` to change into Nginx vhost's log directory using pushd command
* `mypub` to change into Nginx vhost's public web root directory

5. Update Centmin Mod Nginx vhost config file `/usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf` for Matomo nginx rules

Edit your Nginx vhost config file at `/usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf` replacing the default `location / {}` context content

```
  location / {
  include /usr/local/nginx/conf/503include-only.conf;

# block common exploits, sql injections etc
#include /usr/local/nginx/conf/block.conf;

  # Enables directory listings when index file not found
  #autoindex  on;

  # Shows file listing times as local time
  #autoindex_localtime on;

  # Wordpress Permalinks example
  #try_files $uri $uri/ /index.php?q=$uri&$args;

  }
```

with the below content

```
# Matomo Stuff
  # https://github.com/matomo-org/matomo-nginx/blob/master/sites-available/matomo.conf
  add_header Referrer-Policy origin always; # make sure outgoing links don't show the URL to the Matomo instance

  ## only allow accessing the following php files
  location ~ ^/(index|matomo|piwik|js/index|plugins/HeatmapSessionRecording/configs)\.php$ {
    include /usr/local/nginx/conf/php.conf; # we have to include this again here inside of this location block
    try_files $fastcgi_script_name =404; # protects against CVE-2019-11043. If this line is already included in your snippets/fastcgi-php.conf you can comment it here.
    fastcgi_param HTTP_PROXY ""; # prohibit httpoxy: https://httpoxy.org/
  }

  # LOCATION DIRECTIVES
  location / {
    include /usr/local/nginx/conf/503include-only.conf;
    try_files $uri $uri/ =404;

    # Matomo $_SERVER variables
    fastcgi_param MM_COUNTRY_CODE $geoip2_data_country_code;
    fastcgi_param MM_CONTINENT_NAME $geoip2_data_continent_name;
    fastcgi_param MM_COUNTRY_CODE $geoip2_data_country_code;
    fastcgi_param MM_COUNTRY_NAME $geoip2_data_country_name;
    fastcgi_param MM_REGION_CODE $geoip2_data_region_iso;
    fastcgi_param MM_REGION_NAME $geoip2_data_region_name;
    fastcgi_param MM_LATITUDE $geoip2_data_location_latitude;
    fastcgi_param MM_LONGITUDE $geoip2_data_location_longitude;
    fastcgi_param MM_POSTAL_CODE $geoip2_data_postal_code;
    fastcgi_param MM_CITY_NAME $geoip2_data_city_name;
    fastcgi_param MM_ISP $geoip2_data_autonomous_system_organization;
    fastcgi_param MM_ORG $geoip2_data_autonomous_system_number;
  }

  ## disable all access to the following directories
  location ~ ^/(config|tmp|core|lang) {
    deny all;
    return 403; # replace with 404 to not show these directories exist
  }

  location ~ /\.ht {
    deny  all;
    return 403;
  }

  location ~ js/container_.*_preview\.js$ {
    expires off;
    add_header Cache-Control 'private, no-cache, no-store';
  }

  location ~ ^/(libs|vendor|misc|node_modules) {
    deny all;
    return 403;
  }

  location ~ \.(gif|ico|jpg|png|svg|js|css|htm|html|mp3|mp4|wav|ogg|avi|ttf|eot|woff|woff2)$ {
    allow all;
    ## Cache images,CSS,JS and webfonts for an hour
    ## Increasing the duration may improve the load-time, but may cause old files to show after an Matomo upgrade
    expires 1h;
    add_header Pragma public;
    add_header Cache-Control "public";
  }
```

Optionally, you can enable Centmin Mod Nginx JSON formatted access logs for easier parsing of access logs by uncommenting the additional `access_log` line in `/usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf`.

Change from

```
  access_log /home/nginx/domains/analytics.domain.com/log/access.log combined buffer=256k flush=5m;
  #access_log /home/nginx/domains/analytics.domain.com/log/access.json main_json buffer=256k flush=5m;
  error_log /home/nginx/domains/analytics.domain.com/log/error.log;
```

to

```
  access_log /home/nginx/domains/analytics.domain.com/log/access.log combined buffer=256k flush=5m;
  access_log /home/nginx/domains/analytics.domain.com/log/access.json main_json buffer=256k flush=5m;
  error_log /home/nginx/domains/analytics.domain.com/log/error.log;
```

If using Cloudflare, uncomment and enable `/usr/local/nginx/conf/cloudflare.conf` include file in `/usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf`. This ensures Centmin Mod Nginx logs the real visitor IP address.

Change from

```
  # uncomment cloudflare.conf include if using cloudflare for
  # server and/or vhost site
  #include /usr/local/nginx/conf/cloudflare.conf;
```

to

```
  # uncomment cloudflare.conf include if using cloudflare for
  # server and/or vhost site
  include /usr/local/nginx/conf/cloudflare.conf;
```

Comment out `/usr/local/nginx/conf/staticfiles.conf` and `/usr/local/nginx/conf/drop.conf` include files in `/usr/local/nginx/conf/conf.d/analytics.domain.com.ssl.conf`

```
  #include /usr/local/nginx/conf/staticfiles.conf;
  #include /usr/local/nginx/conf/drop.conf;
```
and also comment out Centmin Mod Nginx autoprotect include file `/usr/local/nginx/conf/autoprotect/analytics.domain.com/autoprotect-analytics.domain.com.conf`

```
  #include /usr/local/nginx/conf/autoprotect/analytics.domain.com/autoprotect-analytics.domain.com.conf;
```

Turn off the default `disable_function` for `shell_exec` as Matomo requires the use of it.

```
sed -i 's|^php_admin_value\[disable_functions\]|;php_admin_value\[disable_functions\]|' /usr/local/etc/php-fpm.conf
```

Verify it's disabled with semi-colon in front of the setting:

```
cat /usr/local/etc/php-fpm.conf | grep disable_functions
;php_admin_value[disable_functions] = shell_exec
```

Then restart Nginx using either Centmin Mod command shortcut or systemctl

Restart both Nginx and PHP-FPM for Nginx vhost and PHP-FPM changes to take effect:

```
nprestart
```

or

```
systemctl restart nginx php-fpm
```

For Nginx only restart:

```
ngxrestart
```

or

```
systemctl restart nginx
```

to restart PHP-FPM only:

```
fpmrestart
```

or

```
systemctl restart php-fpm
```

6. Install Matomo

Install Matomo as per guide at https://matomo.org/faq/on-premise/installing-matomo/.

The created Nginx vhost's public web root is at `/home/nginx/domains/analytics.domain.com/public`. I usually like to download files to a staging directory first one level above the public web root and then apply nginx user/group permissions and copy the extracted zip file contents to public web root.

```
mkdir -p /home/nginx/domains/analytics.domain.com/matomo-downloads
# create matomo-archives directory for cron task archive logs
mkdir -p /home/nginx/domains/analytics.domain.com/matomo-archives
cd /home/nginx/domains/analytics.domain.com/matomo-downloads
wget -O /home/nginx/domains/analytics.domain.com/matomo-downloads/matomo.zip https://builds.matomo.org/matomo.zip
unzip matomo.zip
chown -R nginx:nginx matomo
cd matomo
\cp -af * /home/nginx/domains/analytics.domain.com/public
```

Listing public web root's contents after Matomo files copied over:

```
ls -lah /home/nginx/domains/analytics.domain.com/public
total 428K
drwxr-s--- 13 nginx nginx 4.0K Aug  4 14:42 .
drwxr-s---  7 nginx nginx   84 Aug  4 14:39 ..
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 401.html
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 403.html
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 404.html
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 500.html
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 502.html
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 503.html
-rw-r-----  1 nginx nginx 7.6K Aug  4 14:30 503.jpg
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 504.html
-rw-r-----  1 nginx nginx 2.0K Aug  4 14:30 50x.html
-rw-r--r--  1 nginx nginx 109K Jun 10 07:48 CHANGELOG.md
drwxr-sr-x  3 nginx nginx   89 Jun 10 07:48 config
-rwxr-xr-x  1 nginx nginx  753 Jun 10 07:48 console
-rw-r--r--  1 nginx nginx  929 Jun 10 07:48 CONTRIBUTING.md
drwxr-sr-x 53 nginx nginx 4.0K Jun 10 07:48 core
-rw-r--r--  1 nginx nginx  555 Jun 10 07:48 DIObject.php
-rw-r-----  1 nginx nginx 6.3K Aug  4 14:30 index.html
-rw-r--r--  1 nginx nginx  716 Jun 10 07:48 index.php
drwxr-sr-x  2 nginx nginx  114 Jun 10 07:48 js
drwxr-sr-x  2 nginx nginx 4.0K Jun 10 07:48 lang
-rw-r--r--  1 nginx nginx  828 Jun 10 07:48 LegacyAutoloader.php
-rw-r--r--  1 nginx nginx 8.9K Jun 10 07:48 LEGALNOTICE
drwxr-sr-x  9 nginx nginx  140 Jun 10 07:48 libs
-rw-r--r--  1 nginx nginx  35K Jun 10 07:48 LICENSE
-rw-r-----  1 nginx nginx 3.3K Aug  4 14:30 maintenance.html
-rw-r--r--  1 nginx nginx  66K Jun 10 07:48 matomo.js
-rw-r--r--  1 nginx nginx  334 Jun 10 07:48 matomo.php
drwxr-sr-x  7 nginx nginx  127 Jun 10 07:48 misc
drwxr-sr-x 14 nginx nginx  235 Jun 10 07:48 node_modules
-rw-r--r--  1 nginx nginx 6.3K Jun 10 07:48 offline-service-worker.js
-rw-r--r--  1 nginx nginx  66K Jun 10 07:48 piwik.js
-rw-r--r--  1 nginx nginx 2.7K Jun 10 07:48 piwik.php
drwxr-sr-x 69 nginx nginx 4.0K Jun 10 07:48 plugins
-rw-r--r--  1 nginx nginx 4.6K Jun 10 07:48 PRIVACY.md
-rw-r--r--  1 nginx nginx 5.9K Jun 10 07:48 README.md
-rw-r--r--  1 nginx nginx  770 Jun 10 07:48 robots.txt
-rw-r--r--  1 nginx nginx 1.9K Jun 10 07:48 SECURITY.md
drwxr-sr-x  2 nginx nginx   23 Jun 10 07:48 tests
drwxr-sr-x  2 nginx nginx    6 Jun 10 07:48 tmp
drwxr-sr-x 22 nginx nginx  321 Jun 10 07:48 vendor
```

7. Setup Matomo

Open your web browser and navigate to the URL to which you uploaded Matomo - `https://analytics.domain.com`. If everything is uploaded correctly, you should see the Matomo Installation Welcome Screen and proceed as outlined at

8. Setup Matomo set up auto-archiving cron task

Setup Matomo set up auto-archiving cron task as per https://matomo.org/docs/setup-auto-archiving/ with the path to your console at `/home/nginx/domains/analytics.domain.com/public/console` and url to `https://analytics.domain.com` install location

```
5 * * * * /usr/local/bin/php /home/nginx/domains/analytics.domain.com/public/console core:archive --url=https://analytics.domain.com > /home/nginx/domains/analytics.domain.com/matomo-archives/matomo-archive.log
```

You can use `crontab -e` command to open cron editor to add the above line.

9. Optimizations

a. Disable browser triggers for Matomo archiving and limit Matomo reports to updating every hour

After you have set up the automatic archive script as explained above, you can set up Matomo so that requests in the user interface do not trigger archiving, but instead read the pre-archived reports. Login as the super user, click on `Administration > System -> General Settings`, and select:

* Archive reports when viewed from the browser: No
* Archive reports at most every X seconds : 3600 seconds

Click save to save your changes


b. Setup Matomo Tracking API to use a Redis https://matomo.org/faq/on-premise/how-to-configure-matomo-to-handle-unexpected-peak-in-traffic/

  1. Get QueuedTracking from the Marketplace https://plugins.matomo.org/QueuedTracking
  2. Activate the QueuedTracking plugin in “Matomo Administration > Plugins “
  3. Under “Matomo Administration > System > General Settings > QueuedTracking”
  4. Select Backend = Redis and Redis port 6379 & database number = 9 & Select Number of Queue workers = 1 or 2
  5. Select Number of requests that are processed in one batch = 50. This will mean your analytics reported won't be real-time but updated in batches of requests.
  6. Disable the setting Process during tracking request
  7. Then setup a cronjob that executes the command ./console queuedtracking:process every minute, for example:

```
* * * * * php /home/nginx/domains/analytics.domain.com/public/console queuedtracking:process --no-ansi >/dev/null 2>&1
```

Then to check this is working, you can keep track of the queue and see how big it is by executing the command:

```
/home/nginx/domains/analytics.domain.com/public/console queuedtracking:monitor
```

This will show the current state of the queue. In traffic peak time the queue will grow 1,000 or 10,000 requests or more, but usually the queue should be around 0-150 requests.


10. Notes

As Matomo is installed on Nginx, some of the administration (`Diagnostic -> Systems Check`) settings report incorrectly that certain directories or files which are mean to be private are publicly accessible. But these directories and files are already handled by above Nginx vhost's Matomo Nginx rules to deny access.

Required Private Directories that are incorrectly reported as publicly available but are infact 403 permission denied using above Matomo Nginx rules.

```
https://analytics.domain.com/config/config.ini.php
https://analytics.domain.com/tmp/cache/tracker/matomocache_general.php
https://analytics.domain.com/tmp/
https://analytics.domain.com/tmp/empty
https://analytics.domain.com/lang/en.json
```

And file integrity resports Centmin Mod's default Nginx vhost files which should be left available. At least `maintenance.html` should be left available to support Centmin Mod's 503 maintenance mode feature.

```
Files were found in your Matomo, but we didn't expect them.
--> Please delete these files to prevent errors. <--

File to delete: 401.html
File to delete: 403.html
File to delete: 404.html
File to delete: 500.html
File to delete: 502.html
File to delete: 503.html
File to delete: 503.jpg
File to delete: 504.html
File to delete: 50x.html
File to delete: index.html
File to delete: maintenance.html
```

