# Windows Setup

This is a work in progress.  Current requirements are having a working [salt](https://saltproject.io/) server and the salt client installed on the windows system.  All sls files are installed in /srv/salt on the salt master and all windows install/config are in /srv/salt/win.

## Add a management user
  
  win_useradd.sls;

```yaml
admin:
  user.present:
    - fullname: 'admin'
    - createhome: true
    - password: <Cleartext password :( )>

Administrators:
  group.present:
    - system: True
    - addusers:
      - admin

Users:
  group.present:
    - system: True
    - addusers:
      - admin
```

## Enable RDP

win_enable_rdp.sls;

```yaml
enable_rdp:
module.run:
    - name: rdp.enable
```

## Enable ICMP v4

win_enable_icmpv4.sls;

```yaml
icmp_firewall_rule:
  win_firewall.add_rule:
    - name: 'ICMP Allow incoming V4 echo request'
    - dir: in
    - protocol: icmpv4
    - action: allow
    - localport: 1
```

## Turn off standby

win_set_powercfg.sls;

```yaml
disable_standby:
  module.run:
    - powercfg.set_standby_timeout:
      - timeout: 0 
      - power: ac
```

## Install osquery for [Security Onion](https://securityonionsolutions.com/)

*Note: Before installing you need to run so-allow on the security onion box and allow in "Osquery endpoint - 8090/tcp" for either the single client IP or client ip subnet.*

*Note: File is specific to your Security Onion install and can be downloaded from <https://YourSecurityOnionServerIP/#/downloads>*

win_install_kolide_launcher.sls;

```yaml
kolide-install:
  file.managed:
    - name: C:\securityonion\msi-launcher.msi
    - source: salt:///win/msi-launcher.msi
    - makedirs: True
    - replace: True
  cmd.run:
    - name: |
        C:\securityonion\msi-launcher.msi /quiet
```

## Install sysmon

A good sysmon config can be found at <https://github.com/SwiftOnSecurity/sysmon-config>.  Edit for your system and place in /srv/salt/win.

Sysmon64.exe can be downloaded from <https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon>

win_install_sysmon.sls;

```yaml
sysmon-config-updated:
  file.managed:
    - name: C:\securityonion\sysmonconfig-export.xml
    - source: salt:///win/sysmonconfig-export.xml
    - makedirs: True    
    - replace: True
sysmon-install:
  file.managed:
    - name: C:\securityonion\Sysmon64.exe
    - source: salt:///win/Sysmon64.exe
    - makedirs: True
    - replace: True
  cmd.run:
    - name: |
        C:\securityonion\Sysmon64.exe -accepteula -i C:\securityonion\sysmonconfig-export.xml
    - require:
    - sysmon-config-updated    
```

## Install winlogbeats for [Security Onion](https://securityonionsolutions.com/)

*Note: Before installing you need to run so-allow on the security onion box and allow in "Logstash Beat - 5044/tcp" for either the single client IP or client ip subnet.*

*Note: File is specific to your Security Onion install and can be downloaded from <https://YourSecurityOnionServerIP/#/downloads>*

The config file is the standard config file with sysmon added;

```yaml
winlogbeat.event_logs:
- name: Microsoft-Windows-Sysmon/Operational
```

and the security onion server defined;

```yaml
output.logstash:
  # The Logstash hosts
  hosts: ["192.168.1.101:5044"]
```

  win_install_winlogbeat.sls;

```yaml
winlogbeats-install:
  file.managed:
    - name: C:\securityonion\winlogbeat-oss-8.4.3-windows-x86_64.msi
    - source: salt:///win/winlogbeat-oss-8.4.3-windows-x86_64.msi
    - makedirs: True
    - replace: True
  cmd.run:
    - name: |
        C:\securityonion\winlogbeat-oss-8.4.3-windows-x86_64.msi /quiet
winlogbeats-config:
  file.managed:
    - name: C:\ProgramData\Elastic\Beats\winlogbeat\winlogbeat.yml
    - source: salt:///win/winlogbeat.yml
    - replace: True
```

## Install wazuh for [Security Onion](https://securityonionsolutions.com/) (win_install_wazuh.sls)

*Note: Before installing you need to run so-allow on the security onion box and allow in "Wazuh agent - 1514/tcp/udp" and "Wazuh registration service - 1515/tcp" for either the single client IP or client ip subnet.*

*Note: File is specific to your Security Onion install and can be downloaded from <https://YourSecurityOnionServerIP/#/downloads>*

```yaml
wazuh-install:
  file.managed:
    - name: C:\securityonion\wazuh-agent-3.13.1-1.msi
    - source: salt:///win/wazuh-agent-3.13.1-1.msi
    - makedirs: True
    - replace: True
  cmd.run:
    - name: |
        msiexec.exe /i C:\securityonion\wazuh-agent-3.13.1-1.msi /q WAZUH_MANAGER=<ip of your security onion server> WAZUH_REGISTRATION_SERVER=<ip of your security onion server>
```
