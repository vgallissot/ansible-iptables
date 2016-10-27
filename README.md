iptables
========

Installs and configures iptables via netfilter persistent files.

Requirements
------------

This role requires Ansible 2.0 or higher.

Role Variables
--------------

Global vars:
| Name                             | Default                                                                                               | Description                                                             |
|:---------------------------------|:------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------|
| iptables_pkg_install             | yes                                                                                                   | Install packages that allow iptables-restore and iptables-save commands |
| iptables_pkg_state               | present                                                                                               | State of installed package. Either absent, present or latest            |
| iptables_service_state           | started                                                                                               | Manage service state                                                    |
| iptables_service_enabled         | yes                                                                                                   | Start service or not                                                    |
| iptables_provider_ipv4           | iptables                                                                                              | Name used in templates/vars to identify IPv4 specific rules             |
| iptables_provider_ipv6           | ip6tables                                                                                             | Name used in templates/vars to identify IPv6 specific rules             |
| iptables_rules_default_table     | filter                                                                                                | Default table to use in rules definition                                |
| iptables_rules_default_chain     | input                                                                                                 | Default chain to use in rules definition                                |
| iptables_rules_default_target    | accept                                                                                                | Default target to use in rules definition                               |
| iptables_rules_default_provider  | iptables                                                                                              | Default provider to use in rules definition                             |
| iptables_default_policy          | accept                                                                                                | Default policy to use in chains definition                              |
| iptables_tables                  | {}                                                                                                    | Tables's dict you want to handle with ALL chains for each. See examples |
| iptables_rules                   | []                                                                                                    | List of rules. See examples                                             |
| iptables_global_rules            | []                                                                                                    | List of rules merged with iptables_rules. See examples                  |


iptables_tables possible values:
| Name                             | Required | Default                 | Example                                                              | Description                                                             |
|:---------------------------------|:---------|:------------------------|:---------------------------------------------------------------------|:------------------------------------------------------------------------|
| key                              | yes      |                         | {"filter": [],"raw": []}                                             | Table's name                                                            |
| chain                            | yes      |                         | {"key": [{"chain": "input"}]}                                        | Chain's name. ALL chains of each defined tables must be set!            |
| policy                           | no       | iptables_default_policy | {"key": [{"chain": "chain", "policy": "accept"}]}                  | Policy for that chain                                                   |
| provider                         | no       | ALL providers           | {"key": [{"chain": "chain", "provider": "iptables"}]}              | If set, limits current chain config to defined provider                 |


iptables_rules and iptables_global_rules possible values:
| Name      | Required | Default                         | Example                                                                                  | Description                                                             |
|:----------|:---------|:--------------------------------|:-----------------------------------------------------------------------------------------|:------------------------------------------------------------------------|
| name      | yes      |                                 | {"name": "000 Allow everything"}                                                         | Rule's name. Mandatory to ensure rules order.                           |
| in_iface  | no       |                                 | {"name": "001 Allow lo", "in_iface": "lo"}                                               | Name of an interface via which a packet is received                     |
| out_iface | no       |                                 | {"name": "002 Allow out lo", "out_iface": "lo", "chain": "OUTPUT"}                       | Name of an interface via which a packet is sent                         |
| table     | yes      | iptables_rules_default_table    | {"name": "003 Allow all on SNAT", "table": "nat"}                                        | Packet matching table which the command should operate on               |
| chain     | yes      | iptables_rules_default_chain    | {"name": "004 Allow all OUTPUT", "chain": "OUTPUT"}                                      | Append rules to the end of the selected chain                           |
| protocol  | no       |                                 | {"name": "005 Allow all udp", "protocol": "udp"}                                         | The protocol of the rule or of the packet to check                      |
| source    | no       |                                 | {"name": "006 Allow all from home", "source": "1.2.3.4"}                                 | Can be network name, hostname, network IP/mask, or plain IP address     |
| dports    | no       |                                 | {"name": "007 Allow all to 443", "dports": "443", "chain": "OUTPUT"}                     | Destination ports to match. Can be list of ports seperated with dots    |
| sports    | no       |                                 | {"name": "008 Allow all from 80", "dports": "80"}                                        | Source ports to match. Can be list of ports seperated with dots         |
| extras    | no       |                                 | {"name": "009 Allow echo-req", "extras": "-m icmp --icmp-type 8", "protocol": "icmp"}    | Extra iptables expressions not handled here.                            |
| target    | yes      | iptables_rules_default_target   | {"name": "009 Deny all from 80", "dports": "80", "target": "deny"}                       | what to do if the packet matches it                                     |
| provider  | yes      | iptables_rules_default_provider | {"name": "001 Allow lo", "in_iface": "lo", "provider": "ip6tables"}                      | Choose if current rule is either a v4 or v6 rule.                       |


Dependencies
------------

None

Example Playbook
----------------

Install and configure iptables to allow ICMP and OpenSSH
```yaml
- hosts: all
  roles:
    - ansible.iptables
```

Install and configure iptables to disallow ICMP, allow OpenSSH and HTTP
```yaml
- hosts: all
  vars:
    iptables_filter_rules:
      - chain: input
        protocol: tcp
        source_address: 0.0.0.0/0
        destination_port: 22
        comment: OpenSSH
        target: accept
      - chain: input
        protocol: tcp
        source_address: 0.0.0.0/0
        destination_port: 80
        comment: HTTP
        target: accept
  roles:
    - kbrebanov.iptables
```

Install and configure iptables with a port forward rule for HTTP
```yaml
- hosts: all
  vars:
    iptables_filter_rules:
      - chain: input
        protocol: tcp
        source_address: 0.0.0.0/0
        destination_port: 80
        comment: HTTP
        target: accept
    iptables_nat_rules:
      - chain: prerouting
        protocol: tcp
        destination_port: 80
        target: dnat
        to_destination: 192.168.1.54
        to_port: 8080
  roles:
    - kbrebanov.iptables
```

License
-------

BSD

Author Information
------------------

Kevin Brebanov
