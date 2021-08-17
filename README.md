Nftables
========

Dead simple nftables configuration

Requirements
------------

None

Role Variables
--------------

* `nftables_nftables_conf` must contain the raw configuration you want to
  write in `/etc/nftables.conf`. The only added thing that the role does
  is append `include /etc/nftables/` after your statements.

* `nftables_includes` (optional) must be a dictionary. The keys are files
  to be created in `/etc/nftables` (the suffix ".nft" will be appended, so
  a the `rules` will result in the file /etc/nftables/rules.nft). The
  values are the raw content of those files.


Dependencies
------------

None

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: localhost
      roles:
         - chmduquesne.nftables
      vars:
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
          special_rules:
            - |
              # nothing, just a comment



License
-------

BSD
