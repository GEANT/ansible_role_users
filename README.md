Ansible role to configure multiple users accounts.

This takes into account all the user options that ansible supports, minus the SSH related ones.
Instead, this role adds support for the following options:

- `ssh_private_keys`. A list of dictionaries containing a `dest` key (filename below `~/.ssh`), and a `content` key (key material).
- `ssh_authorized_keys`. A list of public keys, these are taken to be 'exclusive'.
- `ssh_config`. An inline snippet of SSH client configuration, saved as `~/.ssh/config`. If you need templating, pick the new option.
- `ssh_config_template`. Same as the above, but this accepts a file name to a template.


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
        # More elaborate example
        user_accounts:
          - name: user4
            group: websvc
            shell: /bin/bash
            password: "{{ 'hackme' | password_hash('sha512', 'mysalt') }}"
            comment: System account for the web services
            system_user: yes
            system_group: yes
            home: /opt/user1
            ssh_authorized_keys:
              - ssh-ed25519 AAAAC3NzaC1lZDI1iweE....
              - ssh-rsa AAAAB3NzaC1yc2EAAAADAQAD....
              - ssh-ed25519 AAAAC3NzaC1lZDIDDJ72....
            ssh_private_keys:
              - dest: git_deploy_key
                content: |
                  -----BEGIN OPENSSH PRIVATE KEY-----
                  b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9u....
                  -----END OPENSSH PRIVATE KEY-----
            ssh_config: |
              Host gitlab.uni.edu
              IdentityFile git_deploy_key
              IdentitiesOnly yes
          - name: hax0r
            state: absent
            remove: yes
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
