ansible-role: dnf-automatic
=========

This role installs, configures and activates `dnf-automatic` via Ansible on hosts which use the dnf package manager. A possible use case is the automatic installation of security updates.

See [https://dnf.readthedocs.org/en/latest/automatic.html](https://dnf.readthedocs.org/en/latest/automatic.html) for more information about `dnf-automatic`.

Requirements
------------

In order for Ansible to work (on Fedora based hosts), it is necessary to have the packages `python2`, `python2-dnf` and `libselinux-python` installed.

Role Variables
--------------

The variable names are mostly self-explanatory. Beside the fact that the names are role-name prefixed, the names are identical to the preferences for the `dnf-automatic` configuration file. See [https://dnf.readthedocs.org/en/latest/automatic.html#configuration-file-format](https://dnf.readthedocs.org/en/latest/automatic.html#configuration-file-format) for details.

In particular, the following variables (including their default values) are used:

```yaml
dnf_automatic_apply_updates: yes
dnf_automatic_download_updates: yes
dnf_automatic_upgrade_type: security
dnf_automatic_random_sleep: 300
dnf_automatic_emit_via: stdio
dnf_automatic_system_name: "{{ ansible_nodename }}"
dnf_automatic_command_format: cat
dnf_automatic_stdin_format: "{body}"
dnf_automatic_email_command_format: mail -s {subject} -r {email_from} {email_to}
dnf_automatic_email_stdin_format: "{body}"
dnf_automatic_email_from: root
dnf_automatic_email_to: root
dnf_automatic_email_host: localhost

dnf_automatic_base_overrides: {}
```

This default configuration sets `dnf-automatic` up to automatically download and install only security updates.

Note that the `dnf_automatic_base_overrides` dictionary can be used to override arbitrary preferences from the base dnf configuration file for `dnf-automatic`.

In addition, `dnf_automatic_reboot` can be set to true to perform automatic reboots when installed updates require it:

```yaml
dnf_automatic_reboot: false
dnf_automatic_reboot_time: "03:00"
dnf_automatic_reboot_script: "/usr/local/sbin/reboot-when-needed.sh"
dnf_automatic_reboot_script_mode: "0700"
dnf_automatic_reboot_script_content: |
  #!/bin/bash
  /bin/needs-restarting -r || /sbin/reboot
dnf_automatic_reboot_dependencies: yum-utils
```

Dependencies
------------

This role has a dependency on `vlcty.systemd-timers` to periodically run the reboot script. It can be installed by adding the following block to roles/requirements.yml:

```
- src: https://github.com/vlcty/ansible-systemd-timers.git
  name: vlcty.systemd-timers
  scm: git
```

Example Playbook
----------------

This example playbook deploys `dnf-automatic` on all hosts but is configured such that all updates are installed automatically, not only security updates.

```yaml
- hosts: all
  remote_user: root
  roles:
  - { role: exploide.dnf-automatic, dnf_automatic_upgrade_type: default }
```

This example playbook deploys `dnf-automatic` to install security updates only, and deploys additional timer to reboot at 4:00 am when required:

```yaml
- hosts: all
  remote_user: root
  roles:
  - { role: exploide.dnf-automatic, dnf_automatic_reboot: true, dnf_automatic_reboot_time: "04:00" }
```


License
-------

MIT
