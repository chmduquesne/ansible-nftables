Nftables
========

Setup nftables

Requirements
------------

None

Role Variables
--------------

* `nftables_nftables_conf_head` must contain the raw configuration you
  want to write at the beginning of `/etc/nftables.conf`. It is advised to
  define it as a group variable and put in there the opening statements
  you want for all hosts. If not specified, it defaults to:

  ```
  flush ruleset
  table inet filter {
    chain input {
      type filter hook input priority 0;
    }
    chain forward {
      type filter hook forward priority 0;
    }
    chain output {
      type filter hook output priority 0;
    }
  }
  ```

* `nftables_nftables_conf_body` must contain the raw configuration you
  want to write in the middle of `/etc/nftables.conf`. It is advised to
  use it for include statements. If not specified, it defaults to:

  ```
  include "/etc/nftables/*.nft"
  ```

* `nftables_nftables_conf_tail` must contain the raw configuration you
  want to write at the end of `/etc/nftables.conf`. It is advised to
  define it as a group variable and put there the closing statements you
  want for all hosts.

* `nftables_includes` (optional) is for creating files in /etc/nftables.
  It is advised to put there host-specific configuration. It must be a
  dictionary. The keys are files to be created in `/etc/nftables` (the
  suffix ".nft" will be appended to the file name) and the values are
  their content. In other words, Using `mykey` as a key and `[mycontent]`
  as its value will result in the creation of the file
  `/etc/nftables/mykey.nft`, with the content `mycontent`.

Exported handlers
-----------------

* The handler `reload nftables` reloads the nftables configuration. It is
  invoked if `/etc/nftables.conf` or any `nftables_include` file is
  modified by this role. You may invoke it in your own playbooks as
  follows:

  ```YAML
  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - reload nftables
  ```

* The handler `restart nftables` restarts nftables. It is not invoked by
  this role, but it is provided for those who want to use it in their own
  playbooks. On debian, restarting will flush the ruleset before having
  nft read `/etc/nftables.conf`, and `reload` will just have nft read
  `/etc/nftables.conf`. Therefore, both achieve exactly the same effect if
  the first line of the file is `flush ruleset`. You may use it as
  follows:

  ```YAML
  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf
    notify:
    - restart nftables
  ```

Dependencies
------------

None

Example Playbook
----------------

Here is roughly of how the author uses the role

```YAML
- hosts: localhost
  roles:
      - chmduquesne.nftables
  vars:
    # This will go at the beginning of /etc/nftables.conf
    nftables_nftables_conf_head:
      - |
        flush ruleset
        table inet filter {
          chain input {
            type filter hook input priority 0; policy drop;
            ct state invalid counter drop comment "drop invalid packets"
            ct state {established, related} counter accept comment "accept all connections related to those we opened"
            iif lo accept comment "accept loopback"
            iif != lo ip daddr 127.0.0.1/8 counter drop comment "drop v4 connections to loopback not coming from loopback"
            iif != lo ip6 daddr ::1/128 counter drop comment "drop v6 connections to loopback not coming from loopback"
            ip protocol icmp counter accept comment "accept ICMP"
            ip6 nexthdr icmpv6 counter accept comment "accept ICMPv6"
            ip protocol udp udp sport 67 udp dport 68 counter accept comment "accept DHCP"
            ip6 nexthdr udp udp sport 547 udp dport 546 counter accept comment "accept DHCPv6"
            tcp dport 22 counter accept comment "accept SSH"
          }
          chain forward {
            type filter hook forward priority 0; policy drop;
          }
          chain output {
            type filter hook output priority 0; policy accept;
          }
        }

    # We do our host-specific customizations in include files
    nftables_includes:
      # will create /etc/nftables/http.nft
      http:
        - |
          add rule inet filter input tcp dport 80 counter accept comment "accept HTTP"
      # will create /etc/nftables/test.nft
      test:
        - |
          add rule inet filter input tcp dport 21 counter accept comment "accept FTP (work in progress)"

    # We explicitly only include the parts that are known to work, so we
    # can safely test the one that are being developed with
    # `nft -f  /etc/nftables/test.nft`
    # without fear of losing the host if we restart the firewall.
    nftables_nftables_conf_body:
      - |
        include "/etc/nftables/http.nft"

    # This will go at the end of /etc/nftables.conf
    nftables_nftables_conf_tail:
      - |
        add rule inet filter input counter accept comment "count dropped packets"
        add rule inet filter forward counter accept comment "count dropped packets"
        add rule inet filter output counter accept comment "count accepted packets"
```

License
-------

BSD
