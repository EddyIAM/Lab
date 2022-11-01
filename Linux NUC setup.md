# Linux NUC Setup

All linux NUCs are initally setup the same via [salt](https://saltproject.io/)

## Add a linux managment user using saltstack

Add user and ssh access.

Prerequisites:  Management system public keys in /srv/salt/pubkeys directory on the salt master.

Filename: lin_useradd.sls

```yaml
admin:
  user.present:
  - fullname: 'admin'
  - createhome: true
  - shell: /bin/bash
  - password: <password is a hash generated with `openssl passwd -1`>
admin_master_sshkey:
  ssh_auth.present:
    - source: salt://pubkeys/admin_master.pub
    - user: admin
sudo:
  group.present:
    - gid: 27
    - system: True
    - addusers:
      - admin
```

## Setup the nfs mount using saltstack

Everybody gets access to a central directory structure.

Prerequisites: NFS setup on the NAS.

Filename: lin_setup_nfs.sls

```yaml
nfs-prereqs:
  pkg.installed:
    - pkgs:
    - nfs-common
create_mount_point:
  file.directory:
    - name: /mnt/docker
mount_dat:
  mount.mounted:
    - name: /mnt/docker
    - device: <NAS_IP>:/docker
    - mkmnt: True
    - fstype: nfs
    - require:
    - file: create_mount_point
```

## Install docker using saltstack

Filename: lin_install_docker.sls

 ```yaml
docker-prereqs:
  pkg.installed:
    - pkgs:
      - apt-transport-https
      - ca-certificates
      - curl
      - gnupg
      - lsb-release
docker-repo:
  cmd.run:
    - name: |
        curl -fsSL https://download.docker.com/ubuntu/ubuntu/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
        echo "deb [arch={{ grains['osarch'] }} signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/ubuntu/ubuntu $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null
        apt update
    - require:
      - docker-prereqs
  pkg.installed:
    - pkgs:
      - docker-ce
      - docker-ce-cli
      - containerd.io
  - require:
    - cmd: docker-repo
  - aggregate: False    
```

## install telegraf using saltstack

Prerequisites:

- InfluxDB and Telegraf docker containers up and running.   [***Example docker-compse.yml***](https://github.com/EddyIAM/Lab-Setup/blob/master/Docker%20setup.md#grafana-influxdb-telegraf-and-prometheus)
- A telegraf.conf file pointing to the influxDB and put in /srv/salt/ubuntu/etc/telegraf
- The influxdb.gpg key put in /srv/salt/ubuntu/etc/apt/trusted.gpg.d

Filename: lin_install_telegraf.sls

```yaml
telegraf-install:
  file.managed:
    - replace: True
    - names: 
      - /etc/apt/trusted.gpg.d/influxdb.gpg:
        - source: salt:///ubuntu/etc/apt/trusted.gpg.d/influxdb.gpg
      - /etc/telegraf/telegraf.conf:
        - source: salt:///ubuntu/etc/telegraf/telegraf.conf
  cmd.run:
    - name: |
        echo 'deb [signed-by=/etc/apt/trusted.gpg.d/influxdb.gpg] https://repos.influxdata.com/debian stable main' | sudo tee /etc/apt/sources.list.d/influxdata.list
        apt update
  pkg.installed:
    - pkgs:
      - telegraf
```

## Install Security Onion version of the OSQuery client using saltstack

*Note: Before installing you need to run so-allow on the security onion box and allow in "Osquery endpoint - 8090/tcp" for either the single client IP or client ip subnet.*

*Note: File is specific to your Security Onion install and can be downloaded from <https://YourSecurityOnionServerIP/#/downloads>*

Filename: lin_install_osquery.sls

```yaml
osquery-install:
  file.managed:
    - name: /securityonion/deb-launcher.deb
    - source: salt:///ubuntu/deb-launcher.deb
    - makedirs: True 
    - replace: True
  cmd.run:
    - name: |
        dpkg -i /securityonion/deb-launcher.deb
```

## Install Security Onion version of Wazuh client using saltstack

*Note: Before installing you need to run so-allow on the security onion box and allow in "Wazuh agent - 1514/tcp/udp" and "Wazuh registration service - 1515/tcp" for either the single client IP or client ip subnet.*

*Note: File is specific to your Security Onion install and can be downloaded from <https://YourSecurityOnionServerIP/#/downloads>*

Filename: lin_install_wazuh.sls

``` yaml
wazuh-install:
file.managed:
  - name: /securityonion/wazuh-agent_3.13.1-1_amd64.deb
  - source: salt:///ubuntu/wazuh-agent_3.13.1-1_amd64.deb
  - makedirs: True
  - replace: True
cmd.run:
  - name: |
      dpkg -i /securityonion/wazuh-agent_3.13.1-1_amd64.deb
      sed -i 's/MANAGER_IP/<Security Onion IP Address>/g' /var/ossec/etc/ossec.conf
      /var/ossec/bin/agent-auth -m <Security Onion IP Address>
      systemctl restart wazuh-agent
```

## Installing velociraptor

1. Setup the velociraptor docker image following [these](https://github.com/weslambert/velociraptor-docker) instructions.  An example docker-compose.yml and .env file can be found [here](https://github.com/EddyIAM/Lab-Setup/blob/master/Docker%20setup.md#velociraptor).

2. Grab the client from wherever you mounted the velociraptor directory at in your docker-compose.yml and put it in your /srv/salt/ubuntu directory.

3. Create the file velociraptor.service in your /srv/salt/ubuntu directory.

    ```text
      [Unit]
      Description=Velociraptor linux amd64
      After=syslog.target network.target
      [Service]
      Type=simple
      Restart=always
      RestartSec=120 
      LimitNOFILE=20000 
      Environment=
      LANG=en_US.UTF-8 
      ExecStart=/usr/local/bin/velociraptor --config /etc/    velociraptor-client.    config.yaml client -v 
      [Install]
      WantedBy=multi-user.target
    ```

4. Push the client via salt.

    filename; lin_install_velociraptor.sls

    ```yaml
      velociraptor-install:
        file.managed:
          - replace: True
          - names: 
            - /usr/local/bin/velociraptor:
              - source: salt://ubuntu/velociraptor_client_repacked
              - mode: 0755
            - /etc/velociraptor-client.config.yaml:
              - source: salt://ubuntu/etc/velociraptor-client.config.yaml
            - /lib/systemd/system/velociraptor.service:
              - source: salt://ubuntu/lib/systemd/system/velociraptor.service

        cmd.run:
          - name: |
              systemctl daemon-reload
              systemctl enable --now velociraptor
    ```
