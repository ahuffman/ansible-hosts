# Hosts

An Ansible role to set a host's hostname and manage it's aliases along with additional host entries.

## Role Variables

You should override this spec at the host_vars level.  These settings apply to a specific host, and the hostname will be set with the Ansible hostname module based on the hosts_hostname.hostname variable.  /etc/hosts will update the loopback IPv4 and IPv6 entries with the appropriate hostname and FQDN from this spec.  Additional IPs can be added with `add_ip` and will add the aliases specified under alias to those host entries.

       `hosts_hostname`:
         `hostname`: ''
         `domain`: ''
         `alias`: {}      #Optional - list of other names you'd like to able to resolve to this IP
         `add_ip`: {}     #Optional - list of additional ips the host you're configuring might have.  This will add all of your listed aliases to the host entry

This spec is for additional hosts you would like defined in your host files and could be used at the group_vars level.  The variables available are quite similar to those above with the exception of `comment` which can add a comment above a host entry.

       `hosts_host`:
          - `name`: ''     #a name for this host entry i.e. 'my db server'
            `hostname`: '' #short hostname i.e. not fqdn
            `domain`: ''   #dns domain of host
            `ip_addr`: ''
            `comment`: ''  #Optional - if you'd like a comment in your host file above this host entry
            `alias`: {}    #Optional - list of other names you'd like to be able to resolve to this IP

## Example Playbook


    /etc/ansible/host_vars/server1/hosts.yml:
        ---
        hosts_hostname:
          hostname: myserver1
          domain: foo.bar.com
          alias:
            - webserver1.foo.bar.com
            - webserver1
            - www.foo.bar.com
            - www
            - db1.foo.bar.com
            - db1
          add_ip:
            - 192.168.122.50
            - 192.168.122.51
            - 192.168.122.52

    /etc/ansible/group_vars/servers/hosts.yml:
         ---
         hosts_host:
           - name: Some Other Webserver
             hostname: myserver2
             domain: foo.bar.com
             ip_addr: 192.168.122.60
             comment: 'The other webserver that does stuff'
             alias:
               - intranet
               - intranet.foo.bar.com
               - helpdesk
               - helpdesk.foo.bar.com

    This will set server1's hostname and configure it's hostfile with the variables passed from host_vars and group_vars
    - hosts: server1
      roles:
         - ahuffman.hosts


## License

[MIT](LICENSE)

## Author Information

[Andrew J. Huffman](https://github.com/ahuffman)
