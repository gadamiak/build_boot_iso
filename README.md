Build a Linux bootable ISO for a bare metal host
================================================

Rebuilds a regular Linux installation ISO image to boot automatically with a
specified kickstart file and include any additional scripts. Supports both
UEFI and legacy BIOS boot modes.

This role, unlike others, copies the source ISO image on the fly, while
modifying boot files and including contents of a specified directory in the
resulting image. It is space efficient and very fast (rebuilding an image
takes <20s).

The assumption is the role is used as a task in a automated deployment, where
each bare metal host gets booted from a dedicated ISO image. The target image
is deleted if exists prior to rebuild, hence you need to deploy hosts one at a
time or customize image file name. It's up to you to provide a kickstart
template and your distribution regular installation image.

Although created for building RHEL images, it can be used with any Linux
distribution by adjusting regex patterns, which modify ISO boot files.

Requirements
------------

- A regular Linux bootable installation image available.
- The `xorriso` package available (installed if missing). On RHEL it is
  available from EPEL repository.

Role Variables
--------------

Variables without defaults required by the role:

```yaml
bbi_iso:
  # The absolute path to a RHEL installation DVD image
  src:
  # The absolute path to the resulting customized image
  dst:
```

Variables in `vars/main.yml`:

```yaml
# Script and kickstart files
bbi_scripts:
  # The full path to the kickstart file in the local filesystem
  ks: ~/kickstart/ks.cfg
  # The absolute path to the kickstart dir in the destination ISO
  path: /kickstart

# Substitutions to be done on files
bbi_regex:
  - # Set default boot menu item for legacy boot
    regexp: '^\s*menu default\s*\n'
    replace: ''
  - # Set default boot menu item for UEFI boot
    regexp: '^(set default=).*'
    replace: '\g<1>"0"'
  - # Set boot menu timeout for legacy boot
    regexp: '(^timeout ).*'
    replace: '\g<1>50'
  - # Set boot menu timeout for UEFI boot
    regexp: '(^set timeout=).*'
    replace: '\g<1>5'
  - # Replace disc label and set kickstart path for legacy boot
    regexp: '(.*)(hd:LABEL=\S+)(.*)'
    replace: '\1\2 inst.ks=\2:{{ scripts.path }}/{{ scripts.ks | basename }}\3'
```

Dependencies
------------

None.

Example Playbook
----------------

The role is delegated to the localhost:

```yaml
- name: Build a customized boot image
  hosts: baremetal
  gather_facts: no

# Some preparation steps
# - download the source ISO
# - prepare kickstart templates
# - install EPEL (RHEL only)

  tasks:

    - name: Build a dedicated bootable ISO image
      import_role:
        name: build_boot_iso
      delegate_to: localhost
      vars:
        bbi_iso:
          src: /var/www/html/rhel-server-7.6-x86_64-dvd.iso
          dst: /var/www/html/rhel-custom.iso

# Some folloup steps
# - mount the ISO image
# - boot the host from the ISO image
# - wait for host to become active
# - configure the host
```

License
-------

GPLv3

Author Information
------------------

Grzegorz Adamiak
