Nftables
========

nftables configuration

Requirements
------------

None

Role Variables
--------------

* `nftables_nftables_conf` must contain the raw configuration you want to
  write in `/etc/nftables.conf`. The only added thing that the role does
  is append `include /etc/nftables/` after your statements.

* `nftables_includes` (optional) must be a dictionary. The keys are files
  to be created in `/etc/nftables` and the values are their content. The
  suffix ".nft" will be appended to the file names. In other words, Using
  `mykey` as a key and `[mycontent]` as its value will result in the
  creation of the file `/etc/nftables/mykey.nft`, with the content
  `mycontent`.

Exported handlers
-----------------

The handler `reload nftables` reloads the nftables configuration. It is
invoked if `/etc/nftables.conf` or any `nftables_include` file (so files
in `/etc/nftables`) is modified by this role. You may invoke it in your
own playbooks as follows:

    - name: Write the apache config file
      ansible.builtin.template:
        src: /srv/httpd.j2
        dest: /etc/httpd.conf
      notify:
      - reload nftables

Dependencies
------------

None

Example Playbook
----------------

Here is an example

    - hosts: localhost
      roles:
         - chmduquesne.nftables
      vars:
        # Use this as the content of /etc/nftables.conf. The line
        # `include /etc/nftables/`
        # is automatically appended at the end of the file.
        nftables_nftables_conf:
          - |
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

        nftables_includes:
          # create /etc/nftables/apache.nft
          apache:
            - |
              # nothing, just a comment

License
-------

BSD
