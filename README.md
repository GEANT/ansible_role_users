Ansible role to configure multiple users accounts.

This takes into account all the user options that ansible supports, minus the SSH related ones.
Instead, this role adds support for the following options:

- `ssh_private_keys`. A list of dictionaries containing a `dest` key (filename below `~/.ssh`), and a `content` key (key material).
- `ssh_authorized_keys`. A list of public keys, these are taken to be 'exclusive'.
  Normally it is only possible to have one `"command"` option per key. If you configure a pubkey entry as a dict with a `pubkey` and `commands` key (list), then a shell wrapper script will be installed as the command, and that script
  in turn will allow all the commands. See below for an example.
- `ssh_config`. An inline snippet of SSH client configuration, saved as `~/.ssh/config`. If you need templating, pick the next option.
- `ssh_config_template`. Same as the above, but this accepts a template file name.


Example playbook
----------------

This examples playbook uses the role twice:


```yaml
---
- hosts: servers

  roles:
    - role: ansible_role_users
      become: true
      vars:
        # Very minimal example
        user_accounts:
          - name: user1
          - name: user2
          - name: user3


    - role: ansible_role_users
      become: true
      vars:
        # Some defaults for all user accounts
        users_user_ssh_authorized_keys_exclusive: yes
        users_user_shell: /bin/dash
        users_user_group: websvc
        users_user_system_user: yes
        users_user_system_group: yes
        # Then the actual users
        user_accounts:
          - name: user4
            password: "{{ 'hackme' | password_hash('sha512', 'mysalt') }}"
            comment: System account for the web services
            home: /opt/user4
            ssh_authorized_keys:
              pubkeys:
                - ssh-ed25519 AAAAC3NzaC1lZDI1iweE....
                - "command=/usr/bin/rsync --foo --bar" ssh-rsa AAAAB3NzaC1yc2EAAAADAQAD....
                - ssh-ed25519 AAAAC3NzaC1lZDIDDJ72....
            ssh_keypairs:
              - dest: git_deploy_key
                private: |
                  -----BEGIN OPENSSH PRIVATE KEY-----
                  b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9u....
                  -----END OPENSSH PRIVATE KEY-----
                public: ssh-rsa AAAAB3NzaC1yc....
            ssh_config: |
              Host gitlab.uni.edu
              IdentityFile git_deploy_key
              IdentitiesOnly yes
          - name: user5
            password: "{{ 'hackmetoo' | password_hash('sha512', 'mysalt') }}"
            comment: Another system account for a web service
            ssh_authorized_keys:
              pubkeys:
                - ssh-ed25519 AAAAC3NzaC1lZDI1iweE....
                - ssh-rsa AAAAB3NzaC1yc2EAAAADAQAD....
                - command="/usr/bin/whoami" ssh-ed25519 AAAAC3NzaC1lZDIDDJ72....
          - name: admin2
            password: "{{ 'hackme2' | password_hash('sha512', 'mysalt') }}"
            comment: Extra admin account with passwordless sudo
            ssh_authorized_keys:
              pubkeys:
                - ssh-ed25519 AAAAC3NzaC1lZDI1iweE....
                # Allow multiple commands for a single key
                - pubkey: ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIAVQU41erftVORV0eYWfBlRjIqDujxzbRsmZ4G0L0xWY
                  commands:
                    - /usr/bin/true
                    - sudo rsync --server --sender -vlogDtprRe.iLsfxCIvu --files-from=- --from0 . /
                    - /usr/bin/whoami
            sudo_config: "ALL=(ALL) NOPASSWD:ALL"
          - name: nagios
            # Useful to pin uid/gid ahead of the software installation
            uid: 110
            gid: 118
```





Optinally you can also configure the currently used account, by setting `name` to `ansible_user`:

```yaml
- hosts: servers

  roles:
    - role: ansible_role_users
      become: true
      vars:
        user_accounts:
          - name: "{{ ansible_user }}"
            ssh_authorized_keys:
              - ssh-ed25519 AAAAC3NzaC1lZDI1iweE....
```

Notes
----

- On a MacOS control machine you need to `pip install password_hash` for the password hashing to work.
- Use only one of `ssh_config` and `ssh_config_template`, as they both write to the same file.


License
-------

BSD

Author Information
------------------

Dick Visser <dick.visser@geant.org>
