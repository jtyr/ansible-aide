aide
====

Ansible role which helps to install and configure AIDE.

The configuration of the role is done in such way that it should not be
necessary to change the role for any kind of configuration. All can be
done either by changing role parameters or by declaring completely new
configuration as a variable. That makes this role absolutely
universal. See the examples below for more details.

Please report any issues or send PR.


Examples
--------

```
---

- name: Example of how to install AIDE with default configuration
  hosts: all
  roles:
    - aide

- name: Example of how to change the configuration
  hosts: all
  vars:
    aide_config_rules__custom:
      # Change rule for /etc/fstab from NORMAL to LSPP
      /etc/fstab: LSPP
      # Add additional rules
      /opt/myapp1: NORMAL
      /opt/myapp2: NORMAL
  roles:
    - aide
```

The configuration doesn't support conditions (`@@undef`, `@@ifdef` and
`@@ifhost`) and includes (`@@include`). The same functionality can be
achieved by with Jinja2 logic in the configuration variable.


Role variables
--------------

```
# Package to be installed (explicit version can be specified here)
aide_pkg: aide

# Location of the config file
aide_config_file: /etc/aide.conf

# Names of the DB and log files
aide_config_database_file_old: aide.db.gz
aide_config_database_file_new: aide.db.new.gz
aide_config_log_file: aide.log

# DB and log permissions
aide_permissions_log_dir_owner: root
aide_permissions_log_dir_group: root
aide_permissions_log_dir_mode: 0700
aide_permissions_db_dir_owner: root
aide_permissions_db_dir_group: root
aide_permissions_db_dir_mode: 0700
aide_permissions_db_file_owner: root
aide_permissions_db_file_group: root
aide_permissions_db_file_mode: 0600


# Whether to setup AIDE DB update cron job
aide_cron: yes

# How often to run the AIDE DB update [hourly|daily|weekly]
aide_cron_time: daily

# Random delay before the AIDE DB update runs from the cron job
aide_cron_job_delay: 0

# Command to run as a cron job for the Clamd Scan
aide_cron_job: sleep $[RANDOM\%{{ aide_cron_job_delay }}]m ; /usr/sbin/aide --check 2>&1 1>{{ aide_config_defines_logdir }}/aide_update_{{ aide_cron_time }}.log


# Default values of the defines
aide_config_defines_dbdir: /var/lib/aide
aide_config_defines_logdir: /var/log/aide

# Default defines
aide_config_defines__default:
  DBDIR: "{{ aide_config_defines_dbdir }}"
  LOGDIR: "{{ aide_config_defines_logdir }}"

# Default defines
aide_config_defines__custom: {}

# Final defines
aide_config_defines: "{{
  aide_config_defines__default.update(aide_config_defines__custom) }}{{
  aide_config_defines__default }}"


# Default values of the configuration parameters
aide_config_database: file:@@{DBDIR}/{{ aide_config_database_file_old }}
aide_config_database_out: file:@@{DBDIR}/{{ aide_config_database_file_new }}
aide_config_gzip_dbout: "yes"
aide_config_verbose: 5
aide_config_report_url:
  - file:@@{LOGDIR}/{{ aide_config_log_file }}
  - stdout

# Default configuration parameters
aide_config__default:
  database: "{{ aide_config_database }}"
  database_out: "{{ aide_config_database_out }}"
  gzip_dbout: "{{ aide_config_gzip_dbout }}"
  verbose: "{{ aide_config_verbose }}"
  report_url: "{{ aide_config_report_url }}"

# Custom configuration parameters
aide_config__custom: {}

# Final configuration parameters
aide_config: "{{
  aide_config__default.update(aide_config__custom) }}{{
  aide_config__default }}"


# Default rules
aide_config_rules__default:
  ALLXTRAHASHES: sha1+rmd160+sha256+sha512+tiger
  EVERYTHING: R+ALLXTRAHASHES
  NORMAL: R+rmd160+sha256
  DIR: p+i+n+u+g+acl+selinux+xattrs
  PERMS: p+i+u+g+acl+selinux
  LOG: ">"
  LSPP: R+sha256
  DATAONLY: p+n+u+g+s+acl+selinux+xattrs+md5+sha256+rmd160+tiger

# Custom rules
aide_config_rules__custom: {}

# Final rules
aide_config_rules: "{{
  aide_config_rules__default.update(aide_config_rules__custom) }}{{
  aide_config_rules__default }}"


# Default directroies/files to observer
aide_config_content__default:
  /bin:                         NORMAL
  /boot:                        NORMAL
  "!/etc/.*~":                  "#"
  /etc/aliases:                 LSPP
  /etc/at.allow:                LSPP
  /etc/at.deny:                 LSPP
  /etc/audit/:                  LSPP
  /etc/bash_completion.d/:      NORMAL
  /etc/bashrc:                  NORMAL
  /etc/cron.allow:              LSPP
  /etc/cron.daily/:             LSPP
  /etc/cron.deny:               LSPP
  /etc/cron.d/:                 LSPP
  /etc/cron.hourly/:            LSPP
  /etc/cron.monthly/:           LSPP
  /etc/cron.weekly/:            LSPP
  /etc/crontab:                 LSPP
  /etc/cups:                    LSPP
  /etc/exports:                 NORMAL
  /etc/fstab:                   NORMAL
  /etc/group:                   NORMAL
  /etc/grub/:                   LSPP
  /etc/gshadow:                 NORMAL
  /etc/hosts.allow:             NORMAL
  /etc/hosts.deny:              NORMAL
  /etc/hosts:                   LSPP
  /etc/inittab:                 LSPP
  /etc/issue:                   LSPP
  /etc/issue.net:               LSPP
  /etc/ld.so.conf:              LSPP
  /etc/libaudit.conf:           LSPP
  /etc/localtime:               LSPP
  /etc/login.defs:              NORMAL
  /etc/logrotate.d:             NORMAL
  /etc/modprobe.conf:           LSPP
  "!/etc/mtab":                 "#"
  /etc/nscd.conf:               NORMAL
  /etc/pam.d:                   LSPP
  /etc/passwd:                  NORMAL
  /etc:                         PERMS
  /etc/postfix:                 LSPP
  /etc/profile.d/:              NORMAL
  /etc/profile:                 NORMAL
  /etc/rc.d:                    LSPP
  /etc/resolv.conf:             DATAONLY
  /etc/securetty:               LSPP
  /etc/security:                LSPP
  /etc/security/opasswd:        NORMAL
  /etc/shadow:                  NORMAL
  /etc/skel:                    NORMAL
  /etc/ssh/ssh_config:          LSPP
  /etc/ssh/sshd_config:         LSPP
  /etc/stunnel:                 LSPP
  /etc/sudoers:                 NORMAL
  /etc/sysconfig:               LSPP
  /etc/sysctl.conf:             LSPP
  /etc/vsftpd.ftpusers:         LSPP
  /etc/vsftpd:                  LSPP
  /etc/X11/:                    NORMAL
  /etc/yum.conf:                NORMAL
  /etc/yumex.conf:              NORMAL
  /etc/yumex.profiles.conf:     NORMAL
  /etc/yum/:                    NORMAL
  /etc/yum.repos.d/:            NORMAL
  /etc/zlogin:                  NORMAL
  /etc/zlogout:                 NORMAL
  /etc/zprofile:                NORMAL
  /etc/zshrc:                   NORMAL
  /lib64:                       NORMAL
  /lib:                         NORMAL
  /opt:                         NORMAL
  /root:                        NORMAL
  /root/\..*:                   PERMS
  /sbin:                        NORMAL
  /usr:                         NORMAL
  /usr/sbin/stunnel:            LSPP
  "!/usr/src":                  "#"
  "!/usr/tmp":                  "#"
  "!/var/log/aide.log":         "#"
  "!/var/log/and-httpd":        "#"
  /var/log/faillog:             LSPP
  /var/log/lastlog:             LSPP
  /var/log:                     LOG
  "!/var/log/sa":               "#"
  /var/run/utmp:                LOG
  /var/spool/at:                LSPP
  /var/spool/cron/root:         LSPP

# Custom directroies/files to observer
aide_config_content__custom: {}

# Final directroies/files to observer
aide_config_content: "{{
  aide_config_content__default.update(aide_config_content__custom) }}{{
  aide_config_content__default }}"
```


Dependencies
------------

- [`config_encoder_filters`](https://github.com/jtyr/ansible-config_encoder_filters)


License
-------

MIT


Author
------

Jiri Tyr
