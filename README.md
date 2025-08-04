[![CI](https://github.com/de-it-krachten/ansible-role-autoinstall/workflows/CI/badge.svg?event=push)](https://github.com/de-it-krachten/ansible-role-autoinstall/actions?query=workflow%3ACI)


# ansible-role-autoinstall

Creates autoinstall file to Ubuntu installation. 



## Dependencies

#### Roles
None

#### Collections
None

## Platforms

Supported platforms

- Red Hat Enterprise Linux 8<sup>1</sup>
- Red Hat Enterprise Linux 10<sup>1</sup>
- RockyLinux 8
- RockyLinux 10
- OracleLinux 8
- OracleLinux 10
- AlmaLinux 8
- AlmaLinux 10
- SUSE Linux Enterprise 15<sup>1</sup>
- Debian 11 (Bullseye)
- Debian 12 (Bookworm)
- Debian 13 (Trixie)
- Ubuntu 20.04 LTS
- Ubuntu 22.04 LTS
- Ubuntu 24.04 LTS
- Fedora 41
- Fedora 42

Note:
<sup>1</sup> : no automated testing is performed on these platforms

## Role Variables
### defaults/main.yml
<pre><code>
# Hostname
autoinstall_hostname: template-ubuntu2204

# Autoinstall distribution
autoinstall_distro: ubuntu2204

# Autoinstall bios mode (uefi or legacy)
autoinstall_bios_mode: uefi

# possible type: fdd, iso, iso-internal, file
autoinstall_format: iso

# Cloudinit location
autoinstall_path: /tmp/{{ autoinstall_hostname }}.iso

autoinstall_packages:
  RedHat:
    - p7zip
    - p7zip-plugins
    - genisoimage
    - xorriso
    - rsync
    - cloud-utils
  Debian:
    - p7zip
    - p7zip-full
    - genisoimage
    - xorriso
    - rsync
    - cloud-image-utils

autoinstall_network:
  hostname: "{{ autoinstall_hostname }}"
  devices:
    - device: "{{ autoinstall_network_device }}"
      dhcp: true
      optional: false
#    - device: ens192
#      ip: [ '192.168.1.100/24' ]
#      optional: false
#      dhcp: false


# Users
autoinstall_users:
  root_password: root
  ansible_user: ansible
  ansible_password: ansible

# Configure LVM storage
autoinstall_configure_storage: true

autoinstall_storage:
  legacy:
    disk:
      - { name: sda, id: disk-sda, lvm: false, grub_device: true }
      # - { name: sdb, id: disk-sdb, lvm: true }
    partition:
      - { name: grub, id: partition-0, device: disk-sda, size: 1M, flag: bios_grub }
      - { name: boot, id: partition-1, device: disk-sda, size: 1G }
      - { name: lvm, id: partition-2, device: disk-sda, size: -1 }
      # - { name: disk2, id: partition-3, device: disk-sdb, size: -1 }
    filesystem:
      - { name: boot, id: format-0, type: ext4, partition: partition-1 }
    lvm:
      vg:
        - { name: rootvg, id: lvm_volgroup-0, devices: [ partition-2 ] }
        # - { name: datavg, id: lvm_volgroup-1, devices: [ partition-3 ] }
      lv:
        - { name: lv_root, id: lvm_partition-0, vg: lvm_volgroup-0, size: 8G }
        - { name: lv_home, id: lvm_partition-1, vg: lvm_volgroup-0, size: 10G }
        - { name: lv_var, id: lvm_partition-2, vg: lvm_volgroup-0, size: 4G }
        - { name: lv_varlog, id: lvm_partition-3, vg: lvm_volgroup-0, size: 5G }
        - { name: lv_usrlocal, id: lvm_partition-4, vg: lvm_volgroup-0, size: 1G }
        - { name: lv_opt, id: lvm_partition-5, vg: lvm_volgroup-0, size: 4G }
        - { name: lv_tmp, id: lvm_partition-6, vg: lvm_volgroup-0, size: 4G }
      fs:
        - { name: lv_root, id: format-3, lv: lvm_partition-0, type: ext4 }
        - { name: lv_home, id: format-4, lv: lvm_partition-1, type: ext4 }
        - { name: lv_var, id: format-5, lv: lvm_partition-2, type: ext4 }
        - { name: lv_varlog, id: format-6, lv: lvm_partition-3, type: ext4 }
        - { name: lv_usrlocal, id: format-7, lv: lvm_partition-4, type: ext4 }
        - { name: lv_opt, id: format-8, lv: lvm_partition-5, type: ext4 }
        - { name: lv_tmp, id: format-9, lv: lvm_partition-6, type: ext4 }
    mount:
      - { path: /, id: mount-3, device: format-3 }
      - { path: /home, id: mount-4, device: format-4 }
      - { path: /var, id: mount-5, device: format-5 }
      - { path: /var/log, id: mount-6, device: format-6 }
      - { path: /usr/local, id: mount-7, device: format-7 }
      - { path: /opt, id: mount-8, device: format-8 }
      - { path: /tmp, id: mount-9, device: format-9 }
      - { path: /boot, id: mount-10, device: format-0 }
  uefi:
    disk:
      - { name: sda, id: disk-sda, lvm: false }
      # - { name: sdb, id: disk-sdb, lvm: true }
    partition:
      - { name: efi, id: partition-0, device: disk-sda, size: 1G, flag: 'boot', grub_device: true }
      - { name: boot, id: partition-1, device: disk-sda, size: 1G }
      - { name: lvm, id: partition-2, device: disk-sda, size: -1 }
    filesystem:
      - { name: boot, id: format-0, type: fat32, partition: partition-0 }
      - { name: boot, id: format-1, type: ext4, partition: partition-1 }
    lvm:
      vg:
        - { name: rootvg, id: lvm_volgroup-0, devices: [ partition-2 ] }
      lv:
        - { name: lv_root, id: lvm_partition-0, vg: lvm_volgroup-0, size: 8G }
        - { name: lv_home, id: lvm_partition-1, vg: lvm_volgroup-0, size: 10G }
        - { name: lv_var, id: lvm_partition-2, vg: lvm_volgroup-0, size: 4G }
        - { name: lv_varlog, id: lvm_partition-3, vg: lvm_volgroup-0, size: 5G }
        - { name: lv_usrlocal, id: lvm_partition-4, vg: lvm_volgroup-0, size: 1G }
        - { name: lv_opt, id: lvm_partition-5, vg: lvm_volgroup-0, size: 4G }
        - { name: lv_tmp, id: lvm_partition-6, vg: lvm_volgroup-0, size: 4G }
      fs:
        - { name: lv_root, id: format-3, lv: lvm_partition-0, type: ext4 }
        - { name: lv_home, id: format-4, lv: lvm_partition-1, type: ext4 }
        - { name: lv_var, id: format-5, lv: lvm_partition-2, type: ext4 }
        - { name: lv_varlog, id: format-6, lv: lvm_partition-3, type: ext4 }
        - { name: lv_usrlocal, id: format-7, lv: lvm_partition-4, type: ext4 }
        - { name: lv_opt, id: format-8, lv: lvm_partition-5, type: ext4 }
        - { name: lv_tmp, id: format-9, lv: lvm_partition-6, type: ext4 }
    mount:
      - { path: /, id: mount-3, device: format-3 }
      - { path: /home, id: mount-4, device: format-4 }
      - { path: /var, id: mount-5, device: format-5 }
      - { path: /var/log, id: mount-6, device: format-6 }
      - { path: /usr/local, id: mount-7, device: format-7 }
      - { path: /opt, id: mount-8, device: format-8 }
      - { path: /tmp, id: mount-9, device: format-9 }
      - { path: /boot, id: mount-10, device: format-1 }
      - { path: /boot/efi, id: mount-11, device: format-0 }


# Create autoinstall enabled vendor ISO
autoinstall_update_vendor_iso: false

# Type of autoinstall vendor iso to create (generic or custom)
autoinstall_iso_type: generic

# Cleanup at exit
autoinstall_cleanup: true

# Temporary path to use
autoinstall_iso_tmpdir: /tmp/toos

# Final command to issue at the end of the installation
autoinstall_final_command: "shutdown -h now"
</pre></code>




## Example Playbook
### molecule/default/converge.yml
<pre><code>
- name: sample playbook for role 'autoinstall'
  hosts: all
  become: 'yes'
  vars:
    autoinstall_hostname: localhost
    autoinstall_network_device: enp0s3
  tasks:
    - name: Include role 'autoinstall'
      ansible.builtin.include_role:
        name: autoinstall
</pre></code>
