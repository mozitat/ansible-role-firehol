# Ansible Role: firehol

Manage iptables firewall using [FireHOL](http://firehol.org/). 
This role is compatible to the debops "framework" and complies with the debops coding policy. 

## Requirements

None, aside from FireHOL is available for your distribution or you install it yourself.

## Dependencies

None.

## Installation

Using ansible galaxy:

```bash
ansible-galaxy install mozitat.firehol
```

## Role Variables

All possible variables and defaults are defined in [`defaults/main.yml`](defaults/main.yml). 
The variable naming and usage follows the debobs project policy. This means there are *inventory level scoped variables* and other roles can use the *firehole__dependent_[interfaces|services|rules]* lists to add rules. 

FireHOL rules work by creating *interfaces* which can have *protection* rules, *policy* definitions, *server* and *client* rules in them.
The *interface* statement binds together firewall rules and network interfaces and there can be more than 1 firehol-*interface* per real interface.   
There are many services predefined, you can define your own services.

The role creates an extra *controller* *interface* with rules for the ansible controllers IP allowing SSH and ping by default. 

For more firehol concepts and especially rule syntax see the [FireHOL Manual](http://firehol.org/firehol-manual/)

### Misc
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__enabled`      | `True`    | Enable or disable FireHOL configuration   |
| `firehol__rpm_url`      | [`rpm url`](https://github.com/firehol/packages/releases/download/2017-02-07-2116/firehol-3.1.2-1.el7.centos.noarch.rpm)    | RPM file for Centos   |
| `firehol__conf_version`      | `6`    | firehol.conf syntax version (ipv6 is enabled)   |

### Settings for `firehol-defaults.conf`
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__load_kernel_modules`      | 1 (0 on containers) | `firehol-defaults.conf` autoload kernel modules   |
| `firehol__ruleset_mode`      | `accurate` | or `optimal`, `firehol-defaults.conf` Ruleset mode   |
| `firehol__drop_invalid`      | `1` | `firehol-defaults.conf` default packet policy, also drops ipv6 NDP. Use protection rules in interfaces to workaround. |
| `firehol__conntrack_helpers_assignment` | `kernel` | or `firehol`, `manual`, `firehol-defaults.conf`, see firehol Documentation    |
| `firehol__log_mode`      | `LOG` | or `ULOG` `NFLOG`, `firehol-defaults.conf` Logging target |
| `firehol__log_options`   | `''` | `firehol-defaults.conf` additional log options like `--log-tcp-options --log-ip-options` |
| `firehol__log_prefix`   | `firehol: ` | `firehol-defaults.conf` log prefix |
| `firehol__log_level`   | `warning` | `firehol-defaults.conf` syslog log level |
| `firehol__log_frequency`   | `1/second` | `firehol-defaults.conf` logging limit |
| `firehol__log_burst`   | `5` | `firehol-defaults.conf` logging limit |

### Ansible Controller Rule
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__ansible_controllers`   | `IP of controller(s)` | The ip address of ansible controller |
| `firehol__ansible_controllers_services`   | `['ssh', 'ping']` | Services for the default controller rule |
| `firehol__ansible_controllers_interface`   | `any` | The real network interface for the controller rule |
| `firehol__ansible_controllers_interface_name`   | `controller` | The name for the ansible controller interface |

### Rule defaults
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__default_interface` | `any` | Default interface used in rules if none is defined |
| `firehol__default_ip_version` | `4` | or `6` `46`, interfaces are created with this version |

### Firehol-*interfaces* for rules
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__[group\|host\|depended_]interfaces` | `[]` | List of dicts with interface definitions, see [source](defaults/main.yml) |
| `firehol__default_interfaces` | `['{controller}', '{any}', '{eth0}'] on min host` | By default 2 "virtual" interfaces and all local network interfaces are added to the list  |

### Services
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__[group\|host\|depended\|default_]services` | `` | list of dicts with service definitions, see [source](defaults/main.yml) |

### Rules
| Variable                | Default |  Comments                                 |
| ----------------------- | ------- | ---------------------------------------- |
| `firehol__[group\|host\|depended\|default_]rules` | `` | list of dicts with rule definitions, see [source](defaults/main.yml). The default rule allows ssh from the ansible controller but denies outbound connections. |

## Example Playbook

```yaml
- hosts: all
  vars:
    firehol__interfaces:
      - name: "local_net"
        realif: "eth0"
        rule_params: ''
        w: 2
        ipv: 4

    firehol__services:
      - name: tinc
        ports: 'tcp/655 udp/655'  # or 'tcp,udp/655[,656]'
        description: Tinc VPN

    firehol__rules:
      - type: policy
        action: RETURN
        interface: 'any'  # lets packets flow to next interface 
      - type: protection
        action: strong
        interface: 'any'
      - type: server
        services: ['ping', 'ssh']
        action: 'accept'
        rule_params: ''
        interface: 'any'
      - type: client
        services: ['dns', 'ping', 'http', 'https']
        action: 'accept'
        rule_params: ''
        interface: 'any'
      
    firehol__host_rules:
      - type: server
        services: ['http', 'https']
        action: 'accept'
        rule_params: 'src 10.0.0.0/24'
        interface: 'local_net'

  roles:
    - role: mozitat.firehol
```

## License

see [LICENSE](LICENSE)

## Author Information

mozit
