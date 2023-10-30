# ansible-haproxy

This role installs and configures HAProxy on hosts and also allows changing the distribution of backend servers of HAProxy backends.

## Variables

| Variable                                   | Required | Default                | Description
|--------------------------------------------|----------|------------------------|------------
| **configuration**
| haproxy_global_stats_socket                | yes      | /var/lib/haproxy/stats | Set socket path
| haproxy_global_stats_socket_level          | no       |                        | Set level of socket. Available options are `user`, `operator`, `admin`. Operator is set by default if nothing is set. [See HAProxy documentation for mor info.](https://www.haproxy.com/documentation/hapee/latest/onepage/#5.1-level)
| haproxy_global_chroot                      | yes      | /var/lib/haproxy       |
| haproxy_global_user                        | yes      | haproxy                | HAProxy User
| haproxy_global_group                       | yes      | haproxy                | HAProxy Group
| haproxy_global_vars                        | no       |                        | List of variables for global configuration part of HAProxy
| haproxy_default_stats                      | yes      | enable                 | Enable or Disable Stats
| haproxy_default_timeout_queue              | yes      | 1m                     |
| haproxy_default_timeout_connect            | yes      | 10s                    |
| haproxy_default_timeout_client             | yes      | 1m                     |
| haproxy_default_timeout_server             | yes      | 1m                     |
| haproxy_default_timeout_check              | yes      | 10s                    |
| haproxy_default_vars                       | no       |                        | List of variables for default configuration part of HAProxy
| haproxy_rsyslog_configuration_file         | no       | RedHat: /etc/rsyslog.d/haproxy.conf<br/> Debian: /etc/rsyslog.d/49-haproxy.conf | rsyslog configuration file for haproxy
| haproxy_rsyslog_module_regex               | no       | RedHat: ^#(\$ModLoad imudp)<br/> Debian: ^#(module\(load="imudp"\)) | regex to activate module imudp
| haproxy_rsyslog_udp_server_regex           | no       | RedHat: ^#(\$UDPServerRun 514)<br/> Debian: ^#(input\(type="imudp" port="514"\)) | regex to activate udp server on port 514
| haproxy_configuration_no_log               | no       | true | Do not show rendered template output on creation
| **userlist configuration**
| haproxy_user_list | no | | List of userlists
| &ensp;name        | yes | | Name of the userlist
| &ensp;users       | yes | | List of users
| &ensp;name        | yes | | Name of the user
| &ensp;pwhash      | yes | | Hashed password from according user. [See here for how to create a hash.](https://www.haproxy.com/documentation/haproxy-configuration-tutorials/authentication/basic-authentication/#hash-passwords-in-the-userlist)
| **haproxy custom configuration**
| haproxy_custom_config | no | | Custom haproxy configuration
| **frontend configuration**
| haproxy_frontends                          | yes      |                        | List of frontends
| &ensp;name                                 | yes      | myHttpFrontend         | Name of the frontend
| &ensp;binds                                | yes      |                        |
| &ensp; &ensp;address                       | yes      | 127.0.0.1              | Frontend address
| &ensp; &ensp;port                          | yes      | 80                     | Frontend port
| &ensp; &ensp;param                         | no       |                        | Frontend params. Eg. "ssl cert /etc/haproxy/ssl"
| &ensp;mode                                 | yes      | http                   | Mode of frontend http, tcp
| &ensp;default_backend                      | yes      | myHttpBackend          | Default backend of frontend
| &ensp;client_timeout                       | yes      | 30s                    |
| &ensp;options                              | no       | httplog                | Options of frontend
| &ensp;vars                                 | no       |                        | List of variables for frontend
| &ensp;comments                             | no       |                        | Comment for frontend
| **backend configuration**
| haproxy_backends                           | yes      |                        |
| &ensp;name                                 | yes      | myHttpBackend          | Name of backend
| &ensp;connect_timeout                      | yes      | 10s                    |
| &ensp;server_timeout                       | yes      | 1m                     |
| &ensp;balance_method                       | no       | roundrobin             | Method for balancing. See [HAProxy documentation for options](https://cbonte.github.io/haproxy-dconv/2.0/configuration.html#balance)
| &ensp;mode                                 | yes      |                        | Mode of backend: http, tcp
| &ensp;servers                              | yes      |                        | List of backend servers
| &ensp; &ensp;name                          | yes      | my-server-1            | Name of backend server
| &ensp; &ensp;address                       | yes      | 192.168.0.3            | IP address of backend server
| &ensp; &ensp;port                          | yes      | 80                     | Port of backend server
| &ensp; &ensp;options                       | no       |                        | Options for backend servers eg. "check cookie a6ae174841dfc53aaede3a29a21242c0"
| &ensp;options                              | no       | httplog, http-check    | Options of backend
| &ensp;aclist                               | no       |                        | Generates Multiple instances of the same ACL with different values (usable e.g for IP-Whitelists)
| &ensp; &ensp;name                     | yes      |                        | Name of the ACL
| &ensp; &ensp;match                    | yes      |                        | Matching method of the item (e.g. `src` or `hdr(X-Forwarded-For)`)
| &ensp; &ensp;vars                     | yes      |                        | Array of the ACL items
| &ensp; &ensp;use_backend              | no       |                        | Define a backend to use when condition for ACL matches.
| &ensp; &ensp; &ensp;name              | yes      |                        | Name of the backend to use.
| &ensp; &ensp; &ensp;operation         | no       | if                     | Operation for condition. Possible values are `if` and `unless`.
| &ensp; &ensp; &ensp;check_acl         | yes      |                        | List of ACLs to check. If a list is given the ACLs will be concatenated with AND
| &ensp;vars                            | no       |                        | List of variables for backend.
| &ensp;comments                        | no       |                        | Comment for backend
| **distribution config/change**
| haproxy_distribution_lb_change             |          | false                  | Adjust distribution of HAProxy backend servers.
| haproxy_distribution_lb_backend_state      | yes      |                        | Set server into state [drain,enabled,disabled]
| haproxy_distribution_lb_backend            | yes      |                        | Name of the backend which should be changed.
| haproxy_distribution_lb_inventory_group    | yes      |                        | Inventory groupname which hold the hosts on which the backend servers should be changed.
| haproxy_distribution_lb_backend_drain_wait | no       | 60                     | Seconds to wait for drain of connections before changing state to MAINT|disabled
| haproxy_distribution_lb_backend_fail_on_not_found | no | false | Fail task if server cant be found in backend
| **maintenance pages**
| haproxy_maintenance_pages_file_path        | no       |                        | If defined, role creates maintenance pages from this path on system

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
