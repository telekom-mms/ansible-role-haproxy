<!-- BEGIN_ANSIBLE_DOCS -->

# Ansible role telekom_mms.haproxy

This role installs and configures HAProxy on hosts and also allows changing the distribution of backend servers of HAProxy backends.

## Supported Operating Systems
- EL
  - all
- Debian
  - all

## Role Variables

- `haproxy_backend_vars`
  - Default: ``
  - Description: List of backend variables
  - Type: list
  - Required: no
- `haproxy_backends`
  - Default: `[{"name": null, "default": "myHttpBackend", "type": "str", "description": "", "connect_timeout": {"default": "10s", "type": "str", "description": ""}, "server_timeout": {"default": "1m", "type": "str", "description": ""}, "mode": {"default": "http", "type": "str", "description": ""}, "servers": [{"name": null, "default": "my-server-1", "type": "str", "description": "", "address": {"default": "192.168.0.3", "type": "str", "description": ""}, "port": {"default": 80, "type": "str", "description": ""}}, {"name": null, "default": "my-server-2", "type": "str", "description": "", "address": {"default": "192.168.0.3", "type": "str", "description": ""}, "port": {"default": 80, "type": "str", "description": ""}}], "options": ["httplog", "http-check"]}]`
  - Description: List of backends
  - Type: list
  - Required: no
- `haproxy_configuration_no_log`
  - Default: `true`
  - Description: Do not show rendered template output on creation
  - Type: bool
  - Required: no
- `haproxy_default_stats`
  - Default: `enable`
  - Description: Enable or disable stats
  - Type: str
  - Required: no
- `haproxy_default_timeout_check`
  - Default: `10s`
  - Description: Set additional check timeout, but only after a connection has been already established.
  - Type: str
  - Required: no
- `haproxy_default_timeout_client`
  - Default: `1m`
  - Description: Set the maximum inactivity time on the client side.
  - Type: str
  - Required: no
- `haproxy_default_timeout_connect`
  - Default: `10s`
  - Description: Set the maximum time to wait for a connection attempt to a server to succeed.
  - Type: str
  - Required: no
- `haproxy_default_timeout_queue`
  - Default: `1m`
  - Description: Set the maximum time to wait in the queue for a connection slot to be free
  - Type: str
  - Required: no
- `haproxy_default_timeout_server`
  - Default: `1m`
  - Description: Set the maximum inactivity time on the server side.
  - Type: str
  - Required: no
- `haproxy_default_vars`
  - Default: ``
  - Description: List of variables for default configuration part of HAProxy
  - Type: list
  - Required: no
- `haproxy_distribution_lb_backend_drain_wait`
  - Default: `60`
  - Description: Seconds to wait for drain of connections before changing state to MAINT|disabled
  - Type: int
  - Required: no
- `haproxy_distribution_lb_change`
  - Default: `false`
  - Description: Adjust distribution of HAProxy backend servers.
  - Type: bool
  - Required: no
- `haproxy_frontend_vars`
  - Default: ``
  - Description: List of frontend variables
  - Type: list
  - Required: no
- `haproxy_frontends`
  - Default: `[{"name": null, "default": "myHttpFrontend", "type": "str", "description": "", "binds": {"default": null, "type": "str", "description": "", "address": {"default": "127.0.0.1", "type": "str", "description": ""}, "port": {"default": 80, "type": "str", "description": ""}}, "mode": {"default": "http", "type": "str", "description": ""}, "default_backend": {"default": "myHttpBackend", "type": "str", "description": ""}, "client_timeout": {"default": "30s", "type": "str", "description": ""}, "options": ["httplog"]}]`
  - Description: List of frontends
  - Type: list
  - Required: no
- `haproxy_global_chroot`
  - Default: `/var/lib/haproxy`
  - Description: Changes current directory to  specified value and performs a chroot() there before dropping privilege
  - Type: str
  - Required: no
- `haproxy_global_group`
  - Default: `haproxy`
  - Description: HaProxy Group
  - Type: str
  - Required: no
- `haproxy_global_stats_socket`
  - Default: `/var/lib/haproxy/stats`
  - Description: Set socket path
  - Type: str
  - Required: no
- `haproxy_global_user`
  - Default: `haproxy`
  - Description: HaProxy User
  - Type: str
  - Required: no
- `haproxy_global_vars`
  - Default: ``
  - Description: List of variables for global configuration part of HAProxy
  - Type: list
  - Required: no
- `haproxy_user_list`
  - Default: ``
  - Description: List of userlists
  - Type: list
  - Required: no

## Dependencies

None.

## Example Playbook

```
- hosts: all
  roles:
    - name: ansible-role-haproxy
```


<!-- END_ANSIBLE_DOCS -->

## Usage

```yaml
---
haproxy_global_vars:
  - "maxconn 4000"
haproxy_default_vars:
  - "maxconn 8000"
  - "option redispatch"
  - "retries 3"

haproxy_user_list:
  - name: myUserList
    users:
      - name: myUser
        pwhash: $5$EWyTNL7RGdz56jUe$uG6dMC02hfCYsuoTbuzFhoc/.Ly4/LxVQaLH3NRdfI6

haproxy_custom_config: |
  listen stats
    bind 0.0.0.0:82
    mode http
    stats enable
    stats uri /
    stats admin if TRUE
    stats realm HAProxy
    stats auth admin:xxxx

haproxy_frontends:
  - name: example-frontend
    binds:
      address: 127.0.0.1
      port: 80
    mode: http
    default_backend: example-backend
    client_timeout: 30s
    options:
      - httplog
    aclist:
      - name: ip_whitelist
        match: src
        vars:
          - 172.29.0.7
          - 172.29.0.5
      - name: "exampleacl"
        match: "req.hdr(Host) -i -m reg"
        vars:
          - "example.com"
        use_backend:
          - name: "example_com_backend"
            check_acl: "exampleacl"
    vars:
      - "# force redirect from http to https"
      - "redirect scheme https if !{ ssl_fc }"
      - "http-request deny if !ip_whitelist"
      - "use_backend another-example-backend if acl_example"
    comments:
      - "# frontend for service xy"

haproxy_backends:
  - name: example-backend
    mode: http
    servers:
      - name: "server1"
        address: "172.29.0.1"
        port: 80
      - name: "server2"
        address: "172.29.0.2"
        port: 80
        options: "backup"
    options:
      - httplog
      - httpchk
    vars:
      - "http-request del-header X-REWRITE"
      - "timeout connect 10s"
      - "timeout server 1m"
    comments:
      - "# backend for service xy"
haproxy_maintenance_pages_file_path: "{{ playbook_dir }}/../files/haproxy"
```


## License

GPLv3

## Author Information

* Andreas Hering
* Daniel Uhlmann
* Christopher Grau
