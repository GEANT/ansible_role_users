Ansible role to configure multiple users accounts.

This takes into account all the user options that ansible supports, minus the SSH generation ones.
Instead, the account can have SSH related options, see below playbook.


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





Optinally you can also configure the currently used account, by setting `name` to `ansible_user_id`:

```yaml
- hosts: servers

  roles:
    - role: ansible_role_users
      become: true
      vars:
        user_accounts:
          - name: "{{ ansible_user_id }}"
            ssh_authorized_keys:
              - ssh-ed25519 AAAAC3NzaC1lZDI1iweE....
```

Notes
----

- The role should be run with `--become`, but the main playbook should be run _without_ `--become`.
- On a MacOS control machine you need to `pip install password_hash` for the password hashing to work.



License
-------

BSD

Author Information
------------------

Dick Visser <dick.visser@geant.org>
