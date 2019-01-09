# ahuffman.hosts

An Ansible role to set a host's hostname and manage /etc/hosts entries.

## Role Variables
|Variable Name|Description|Required|Default Value|Type|
|---|---|:---:|:---:|:---:|
|hosts_hostname|Defines the system's hostname, and manages its /etc/hosts entries.|no|{}|dictionary|
|hosts_add_hosts|Defines additional /etc/hosts entries.|no|[{}]|list of dictionaries|
|hosts_backup|Backup /etc/hosts when changing the file.|no|True|boolean|
|hosts_set_hostname|Set hostname on system. Set False to only manage /etc/hosts|no|True|boolean|

***if you do not set hosts_hostname (not required) a default ipv4 loopback line will be created in /etc/hosts if hosts_add_hosts is defined, but ipv6 loopback will not get created***

## `hosts_hostname` Dictionary Fields
|Field Name|Description|Required|Default Value|Type|
|---|---|:---:|:---:|:---:|
|hostname|Short (non-fqdn) hostname of the server|yes|n/a|string|
|domain|Domain name of the server|yes|n/a|string|
|alias|Additional aliases the server is known by (cnames)|no|n/a|list|
|alias_loopback|Whether or not to add the hostname and aliases to the /etc/hosts loopback line(s).|yes|n/a|boolean|
|add_ips|Additional IP addresses for the server in /etc/hosts.|no|n/a|list|
|ipv6|Whether IPv6 is used on the system (sets IPv6 loopback line)|yes|n/a|boolean|

### `hosts_hostname` Usage Example
Here is an example of using the hosts_hostname dictionary in a variable file:
```yaml
---
hosts_hostname:
  hostname: "foo"
  domain: "bar.com"
  alias:
    - "web1"
    - "intranet"
  alias_loopback: False # Don't modify localhost loopback in /etc/hosts
  add_ips:
    - "192.168.122.100" # main NIC
    - "10.10.10.100" # another NIC
    # you could add ipv6 addresses to this list as well
  ipv6: False # Don't create ipv6 loopback entries  
```
This will produce lines in /etc/hosts as such:
```
127.0.0.1        localhost localhost.localdomain localhost4 localhost4.localdomain4
192.168.122.100  foo.bar.com foo web1 intranet
10.10.10.100     foo.bar.com foo web1 intranet
```
## `hosts_add_hosts` Dictionary Fields
|Field Name|Description|Required|Default Value|Type|
|---|---|:---:|:---:|:---:|
|hostname|Short (non-fqdn) hostname of the server|yes|n/a|string|
|domain|Domain name of the server|yes|n/a|string|
|ip_addr|IP address of the server|yes|n/a|string|
|comment|Comment line to put above entry in /etc/hosts|no|n/a|string|
|alias|Additional aliases the server is known by (cnames)|no|n/a|list|

### `hosts_add_hosts` Usage Example
Here is an example of using the hosts_hostname dictionary in a variable file:
```yaml
hosts_add_hosts:
  - hostname: "otherserver1"
    domain: "bar.net"
    ip_addr: "10.100.5.66"
    comment: "External database server"
    alias:
      - "db1"
      - "database"
      - "db"
  - hostname: "anotherserver"
    domain: "coolsite.com"
    ip_addr: "1.2.3.4"
    comment: "A cool website"
```
This will produce lines in /etc/hosts as such:
```
# External database server
10.100.5.66      otherserver1.bar.net otherserver1 db1 database db
# A cool website
1.2.3.4          anotherserver.coolsite.com anotherserver
```
## Example Playbook
host_vars/myserver1.yml contents:
```yaml
---
# Set host specific hostname and /etc/hosts entries
hosts_hostname:
  hostname: "myserver1"
  domain: "foo.bar.com"
  alias:
    - "webserver1.someotherdomain.com"
    - "www"
    - "db1"
    - "dubdubdub.dubs.com"
  alias_loopback: False
  add_ips:
    - "192.168.122.50"
    - "192.168.122.51"
    - "192.168.122.52"
  ipv6: True
```
```yaml
---
- name: "Set hostname and /etc/hosts entries"
  hosts: "all"
  tasks:
    - name: "Ensure hostname and /etc/hosts are in desired state"
      include_role:
        name: "ahuffman.hosts"
      vars:
        hosts_backup: False
        hosts_set_hostname: True
        hosts_add_hosts:
          - hostname: "myserver2"
            domain: "foo.bar.com"
            ip_addr: "192.168.122.60"
            comment: 'The other webserver that does stuff'
            alias:
              - "intranet.externaldomain.com"
              - "intranet"
              - "helpdesk.helpme.com"
              - "helpdesk"
          - name: "Another App server"
            hostname: "appserver2"
            domain: "bar.com"
            ip_addr: "192.168.122.70"
            comment: "App server 1"
```
The /etc/hosts file produced by the above example would look like:
```
# Ansible managed
127.0.0.1        localhost localhost.localdomain localhost4 localhost4.localdomain4
::1              localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.122.50   myserver1.foo.bar.com myserver1 webserver1.someotherdomain.com www db1 dubdubdub.dubs.com
192.168.122.51   myserver1.foo.bar.com myserver1 webserver1.someotherdomain.com www db1 dubdubdub.dubs.com
192.168.122.52   myserver1.foo.bar.com myserver1 webserver1.someotherdomain.com www db1 dubdubdub.dubs.com

# The other webserver that does stuff
192.168.122.60   myserver2.foo.bar.com myserver2 intranet.externaldomain.com intranet helpdesk.helpme.com helpdesk
# App server 1
192.168.122.70   appserver2.bar.com appserver2
```
## License
[MIT](LICENSE)

## Author
[Andrew J. Huffman](https://github.com/ahuffman)
