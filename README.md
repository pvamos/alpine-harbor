# Deploy Harbor registry on Alpine linux with Ansible, generating CA and server keys and certificates
 

<https://github.com/pvamos/alpine-harbor>

A quick ansible playbook to deploy a single Harbor open source registry <https://goharbor.io/> instance,
on a clean Alpine linux machine <https://alpinelinux.org/>,
also upgrading Alpine linux to `latest-stable` (tested on Alpine 3.16, 3.17, and 3.18).

Also generating CA and server keys and certificates, and configuring them,
as described in the official Harbor documentation <https://goharbor.io/docs/2.8.0/install-config/>.

## Requirements:

  Alpine linux 3.16 or newer (playbook updates to latest stable)
    With configured:
      - keyless ssh access with passwordless sudo rights (I use Dropbear)
      - bash shell
      - python 3

  2 CPU cores, 4GB RAM, 40GB disk in official Harbor documentation.
  <https://goharbor.io/docs/2.8.0/install-config/installation-prereqs/>

  A small nonprod/lab/dev deployment works stable on 1 CPU core, 2GB RAM, 20GB disk.

  Tested on an OVHcloud `starter` VPS model `VPS vps2020-starter-1-2-20` <https://www.ovhcloud.com/en-ie/vps/>
    1 vCore
    2 GB
    20 GB SSD SATA
    running on a 2.4 GHz Intel Haswell CPU according to `/proc/cpuinfo`


## Configuration parameters:

   Set in `group_vars/all` group variables file. Template provided at `group_vars/all.template`. (copy group_vars/all.template group_vars/all)
     - ansible_port: PORT
     - ansible_user: USERNAME
     - ansible_ssh_private_key_file: ~/.ssh/PRIVATEKEYFILE
     - reg_fqdn: hostname.example.com
     - reg_hostname: hostname
     - reg_startpassword: NotVerySecurePassword

   Set FQDN in `hosts` file. Template provided in `hosts/template`. (copy hosts/template hosts)

## Main ansible playbook:

  alpine-harbor.yaml

  It follows and completes the below steps described in the official Harbor documentation:

    - Download the Harbor Installer
        <https://goharbor.io/docs/2.8.0/install-config/download-installer/>
    - Configure HTTPS Access to Harbor
       <https://goharbor.io/docs/2.8.0/install-config/configure-https/>
    - Configure the Harbor YML File
        <https://goharbor.io/docs/2.8.0/install-config/configure-yml-file/>
    - Run the Installer Script
        <https://goharbor.io/docs/2.8.0/install-config/run-installer-script/>


## Ansible roles (in order of execution):

    - alpine_latest
        Set `/etc/apk/repositories` to `latest/main`, `latest/community` and `latest/testing`
        Upgrade all installed apk packages to the latest versions
        Reboot the host after apk upgrade

    - alpine_kernelparams
        Set net forwarding kernel params:
          ```
          net.ipv4.ip_forward = 1
          net.ipv6.conf.all.forwarding = 1
          net.bridge.bridge-nf-call-iptables = 1
          net.bridge.bridge-nf-call-ip6tables = 1
          ```
        Enable modules:
          ```
          br_netfilter
          iptable_nat
          iptable_filter
          ip6table_nat
          ip6table_filter
          nf_conntrack
          overlay
          ```
        Configure cgroups version 2 mode to mount cgroups
        Reboot the host after kernel parameter changes

    - alpine_harbor_binaries
        Install packages python3 py-setuptools tar gzip sed curl bash openssl docker docker-compose gpg ncurses
        Download <https://github.com/goharbor/harbor/releases/download/v2.8.3/harbor-online-installer-v2.8.3.tgz>
          and validate it with GPG and published Harbor signing public key
        Create group and user for Harbor
          group: `harbor`
          GID: `10000`
          user: `harbor`
          UID: `10000`
        Create `/harbor` directory if it does not exist
        Generate CA certificate private key
        Generate CA certificate
        Generate Server Certificate private key
        Generate certificate signing request
        Create x509 v3 extension file
        Use the x509 v3 extension file to generate Harbor host certificate
        Create `/data/cert` directory if it does not exist
        Copy Harbor host certificate to `/data/cert`
        Convert .crt to .cert, for use by Docker
        Create `/etc/docker/certs.d/{FQDN}`
        Copy .cert .key and CA .crt to `/etc/docker/certs.d/{FQDN}`
        Add docker service to default runlevel
        Start docker service
        Check docker service status

    - alpine_harbor_config
        Create configuration file `/harbor/harbor.yml`
        Run `/harbor/prepare`
        Run `/harbor/install.sh --with-trivy`


## Improvement possibilities (TODO):

  - All parameters to group_vars/all group variables (remove "burnt in" values, provide a general config template)
  - Implement status checks after completed steps
  - Enabling Internal TLS
      <https://goharbor.io/docs/2.8.0/install-config/configure-internal-tls/>
  - Publish and reference the repo here for Alpine linux basic setup on OVHcloud VPS
  - Backup
  - Upgrade handling
  - Monitoring


---

created by Péter Vámos pvamos@gmail.com <https://www.linkedin.com/in/pvamos>

---

MIT License

Copyright (c) 2023 Péter Vámos

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
