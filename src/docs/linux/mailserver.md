# Mail Server - Postfix, Dovecot, PostfixAdmin, Amavis, ClamAv

## Introduction

I have been running my own mail server for more than 10 years. I have never replaced the services I use.

The following services are used:

* Postfix (mail server)
* Dovecot (IMAP service)
* Amavis filter with ClamAv and Spamassassin
* PostfixAdmin (manage domains/mailboxes)
* Roundcube webmail (browser-based IMAP client)

When upgrading from Debian bookworm to trixie, Dovecot was also updated to version 2.4.1. This update was extremely challenging, so I am documenting my configuration here.

## Dovecot

With the update from Dovecot 2.3 to 2.4, there were many breaking changes that required converting the configuration to the new format.

The configuration is limited to a single file, and the provided configuration file `/etc/dovecot/dovecot.conf` is explicitly overwritten, rather than following the recommendation to put custom changes in `local.conf`. This is to prevent default modules or protocols from being loaded from the standard configuration.
The listings for the Sieve filters and scripts follow after the Dovecot configuration.

This setup is for virtual mailboxes. The virtual user and group `vmail` was created with `uid=150` and `gid=5000`.

### Dovecot configuration file

```ini title="/etc/dovecot/dovecot.conf"
# dovecot 2.4 config

# Following configuration must be first parameter in config file
dovecot_config_version = 2.4.1
dovecot_storage_version = 2.4.1

## Debug stuff
#auth_verbose = yes
#auth_verbose_passwords = no
#auth_debug = yes
#auth_debug_passwords = yes
#mail_debug = yes
#sieve_trace_debug = yes
#sieve_trace_level = matching

# The default ssl_cipher_list is sufficient; no need to override.
# ssl_cipher_list = ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA
# Explicitly set ssl_min_protocol to TLSv1.3 (default is TLSv1.2)
ssl_min_protocol = TLSv1.3
# prefer server ciphers
ssl_server_prefer_ciphers = server
# SSL/TLS is required for all imap, pop3, managesieve and submission protocol client connections. 
ssl = required

ssl_server_key_file = /etc/custom/certs/main_domain/key.pem
ssl_server_cert_file = /etc/custom/certs/main_domain/fullchain.pem

# If you have multiple domains you need to create a separate local_name block for each domain
local_name second_domain {
    ssl_server_key_file = /etc/custom/certs/second_domain//key.pem
    ssl_server_cert_file = /etc/custom/certs/second_domain/fullchain.pem
}

listen = *, ::
auth_cache_size = 1M
auth_cache_ttl = 10s
mailbox_list_index = yes
maildir_very_dirty_syncs = yes
log_timestamp = "%Y-%m-%d %H:%M:%S "
mail_max_userip_connections = 50
first_valid_uid = 150
last_valid_uid = 150

auth_mechanisms = plain login
auth_username_format = %{user}
auth_username_chars = abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ01234567890.-_@

# Driver and fs location of mails
mail_driver = maildir
# home path is set via userdb/passdb -> see userdb_home field in query
mail_path = ~/Maildir
mailbox_list_layout = fs

## do the database stuff here
passdb_default_password_scheme = SHA512-CRYPT
sql_driver = mysql
# local postfixadmin db
mysql localhost {
    user = dbuser
    password = dbpass
    dbname = postfixadmin
    host = 127.0.0.1
}

passdb sql {
    query =     SELECT username as user, \
                password, \
                '/var/vmail/%{user|domain}/%{user|username}' as userdb_home, \
                150 as userdb_uid, \
                5000 as userdb_gid, \
                CONCAT(CAST(mailbox.quota AS CHAR), 'B') AS  userdb_quota_storage_size, \
                crypt as userdb_mail_crypt_save_version, \
                TO_BASE64('%{password}') AS userdb_mail_crypt_private_password \
                FROM mailbox, domain \
                WHERE username = '%{user}' AND mailbox.active = '1' AND domain.active = '1' AND domain.domain = '%{user|domain}'
}

userdb sql {
    query =        SELECT '/var/vmail/%{user|domain}/%{user|username}' as home, \
                150 AS uid, \
                5000 AS gid, \
                crypt AS mail_crypt_save_version, \
                TO_BASE64('%{password}') AS mail_crypt_private_password, \
                CONCAT(CAST(mailbox.quota AS CHAR), 'B') AS  quota_storage_size \
                FROM mailbox, domain \
                WHERE username = '%{user}' AND mailbox.active = '1' AND domain.active = '1' AND domain.domain = '%{user|domain}'
}

## end database stuff

## mailcrypt plugin - not used for now
# db queries include needed fields
#crypt_user_key_curve = brainpoolP160r1
#mail_crypt_save_version = 2
#crypt_user_key_require_encrypted = yes


mail_attribute {
    dict file {
        path = %{home}/Maildir/dovecot-attributes
    }
}

# enable plugins global
mail_plugins {
    quota = yes
    acl = yes
    mail_log = yes
    notify = yes
}

namespace inbox {
    type = private
    inbox = yes
    separator = /
    
    mailbox Trash {
        auto = subscribe
        special_use = \Trash
        quota_storage_percentage = 110
    }
    
    mailbox "Deleted Messages" {
        special_use = \Trash
    }
    
    mailbox "Deleted Items" {
        special_use = \Trash
    }
    
    mailbox "Gelöschte Objekte" {
        special_use = \Trash
    }

    mailbox "Gelöschte Elemente" {
        special_use = \Trash
    }
    
    mailbox "Papierkorb" {
        special_use = \Trash
        quota_storage_percentage = 110
    }
    
    mailbox Archive {
        auto = subscribe
        special_use = \Archive
    }
    
    mailbox Archiv {
        special_use = \Archive
    }
    
    mailbox Sent {
        auto = subscribe
        special_use = \Sent
        quota_storage_percentage = 110
    }
    
    mailbox "Sent Messages" {
        special_use = \Sent
    }
    
    mailbox "Sent Items" {
        special_use = \Sent
    }
        
    mailbox "Gesendet" {
        special_use = \Sent
    }
    
    mailbox "Gesendete Elemente" {
        special_use = \Sent
    }
        
    mailbox "Gesendete Objekte" {
        special_use = \Sent
    }
    
    mailbox Drafts {
        auto = subscribe
        special_use = \Drafts
    }
    
    mailbox Entwürfe {
        special_use = \Drafts
    }

    mailbox Junk {
        auto = subscribe
        special_use = \Junk
    }
    
    mailbox "Junk E-mail"{
        special_use = \Junk
    }
    
    mailbox Spam {
        special_use = \Junk
    }
    
    prefix =
}

# mail_log plugin
mail_log_events = delete undelete expunge

## Begin: user-to-user user shared mailbox-folders
# Ensure both `acl` (global) and `imap_acl` (protocol imap) plugins are enabled in the `mail_plugins` section
# for shared mailbox functionality.
acl_sharing_map {
    dict file {
        path = /var/vmail/shared-mailboxes.db
    }
}

#In each mailbox, Dovecot maintains a file that manages the sharing permissions
acl_driver = vfile

namespace usershares {
    type = shared
    separator = /
    prefix = Geteilt/$user/
    mail_path = %{owner_home}/Maildir
    mail_index_private_path = %{owner_home}/shared/%{user}
    mailbox_list_layout = fs
    list = children
    subscriptions = yes
    hidden = no
}
## End: user-to-user user shared mailbox-folders

service dict {
    unix_listener dict {
        mode = 0660
        user = vmail
        group = vmail
    }
}

service auth {
    unix_listener /var/spool/postfix/private/auth_dovecot {
        group = postfix
        mode = 0660
        user = postfix
    }
    
    unix_listener auth-master {
        mode = 0600
        user = vmail
    }
    
    unix_listener auth-userdb {
        mode = 0600
        user = vmail
    }
}

service managesieve-login {
    inet_listener sieve {
        port = 4190
    }
    service_restart_request_count = 1
    process_min_avail = 2
    vsz_limit = 128M
}

service managesieve {
    process_limit = 256
}

service lmtp {
    unix_listener /var/spool/postfix/private/dovecot-lmtp {
        group = postfix
        mode = 0600
        user = postfix
    }
    user = vmail
}

protocols {
    lmtp = yes
    imap = yes
    sieve = yes
}

protocol imap {
    mail_plugins {
        imap_quota = yes
        imap_acl = yes
        imap_sieve = yes
        imap_filter_sieve = yes
    }
    mail_max_userip_connections = 20
    imap_idle_notify_interval = 29 mins
}

protocol lmtp {
    mail_plugins {
        sieve = yes
    }
    auth_socket_path = /var/run/dovecot/auth-master
    postmaster_address = postmaster@main_domain
}

# Directory containing scripts or binaries that can be executed by the vnd.dovecot.pipe Sieve extension.
# Ensure that only trusted and properly permissioned binaries are placed here, as scripts in this directory
# may be executed with the privileges of the Dovecot process. Restrict write access to prevent unauthorized modifications.
sieve_pipe_bin_dir = /var/vmail/sieve_pipe_bin

sieve_plugins {
      sieve_imapsieve = yes
      # needed for vnd.dovecot.pipe
      sieve_extprograms = yes
}

protocol sieve {
    managesieve_logout_format = bytes=%{input}/%{output}
}

sieve_global_extensions {
    # The "vnd.dovecot.pipe" extension must be activated separately; enabling sieve_extensions is not enough.
    vnd.dovecot.pipe = yes
    vnd.dovecot.environment = yes
}

# user sieve scripts
sieve_script personal {
    path = ~/sieve
      active_path = ~/sieve/dovecot.sieve
}

# global sieve script before user sieve scripts will be executed
sieve_script before-script { 
    type = before 
     cause = delivery 
     path = /var/vmail/prefiltering.sieve
}

protocol lda {
  mail_plugins {
    sieve = yes
  }
}

## begin autolearn spam/ham with spamassassin via amavis
mailbox Junk {
      # From elsewhere to Spam folder
      sieve_script report-spam {
        type = before
        cause = copy
        path = /var/vmail/sieve_spam_handling/report-spam.sieve
      }
}

# From Spam folder to elsewhere
imapsieve_from Junk {
      sieve_script report-ham {
        type = before
        cause = copy
        path = /var/vmail/sieve_spam_handling/report-ham.sieve
      }
}
## end autolearn spam/ham with spamassassin via amavis

# Global quota - will be overwritten with userdb field in query
quota_storage_size = 1G

#Quota for users
quota "User quota" {
}

#Quota for domains
quota "Domain quota" {
}

# Continue even if the quota cannot be determined
# Applies to the Postfix policy service provided by Dovecot
quota_status_success = DUNNO
quota_status_nouser = DUNNO
quota_status_overquota = "552 5.2.2 Mailbox is over quota"

service quota-status {
    executable = quota-status -p postfix
    inet_listener postfix {
        port = 12340
    }
    client_limit = 1
}

service imap-login {
    inet_listener imap {
        port=0
    }
}

service dict {
    unix_listener dict {
        mode = 0600
        user = vmail
    }
}

service anvil {
    # Socket permissions are modified here to ensure the correct group access for Dovecot services.
      unix_listener anvil {
        group = dovecot
        mode = 0660
      }
}

```

### Sieve filter

Global sieve script for moving messages, which was tagged by SpamAssassin as spam, to Junk folder.

```bash title="/var/vmail/prefiltering.sieve"
require "fileinto";
if header :contains "X-Spam-Flag" "YES" {
    fileinto "Junk";
}
```

Sieve filter for SpamAssassin auto-learning.  
This filter trigger the following scripts when emails are marked as spam in the mail client (moved to the Junk folder) or when emails are moved from the Junk folder to another folder (except Trash).

```bash title="/var/vmail/sieve_spam_handling/report-spam.sieve"
require ["vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"];

if environment :matches "imap.user" "*" {
  set "username" "${1}";
}

pipe :copy "sa-learn-spam.sh" [ "${username}" ];
```

```bash title="/var/vmail/sieve_spam_handling/report-ham.sieve"
require ["vnd.dovecot.pipe", "copy", "imapsieve", "environment", "variables"];

if environment :matches "imap.mailbox" "*" {
    set "mailbox" "${1}";
}

if string "${mailbox}" "Trash" {
    stop;
}

if environment :matches "imap.user" "*" {
    set "username" "${1}";
}

pipe :copy "sa-learn-ham.sh" [ "${username}" ];
```

### Scripts for piped sieve filter

!!! info "User `vmail` must be allowed to sudo command sa-learn as user `amavis`"
    Spamassassin is running via Amavis, so user must be `amavis` to globally train Spamassassin

    Edit sudoer file:
    
    `vmail   ALL= (amavis) NOPASSWD: /usr/bin/sa-learn`

```bash title="/var/vmail/sieve_pipe_bin/sa-learn-spam.sh"
#!/bin/sh
sudo -u amavis sa-learn --spam
```

```bash title="/var/vmail/sieve_pipe_bin/sa-learn-ham.sh"
#!/bin/sh
sudo -u amavis sa-learn --ham
```

## Postfix

`ToDo`

## Amavis

`ToDo`

## PostfixAdmin

`ToDo`
