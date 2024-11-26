# Quickstart - Manage AIX with Ansible

Quickstart guide to setting up Ansible between two servers using the same non-privileged user.

```
+-------------+                  +--------------+
|   Ansible   |      ssh/22      |   Ansible    |
|  Execution  +----------------->|   managed    |
| Environment |                  |     AIX      |
+-------------+                  +--------------+
```

## Setup the environment

Install Ansible on the first server to use it as Ansible execution environment.

```dnf install ansible```

Check the ansible installation and installed version.

```ansible --version```

Prepare a SSH key pair for password less SSH connections to Ansible managed systems.

```ssh-keygen -t ed25519 -C "Ansible"```

View your generated SSH public key and copy it for futher steps.

```cat ~/.ssh/id_ed25519.pub```

Add your prepared SSH public key to your authorized_keys on the Ansible managed AIX.

```echo "yourgeneratedpublickey" >> ~/.ssh/authorized_keys```

Create a simple inventory with the group all and the DNS name of yourmanagedAnsibleAIX.

```bash
echo "[all]" >> inventory.ini
echo "yourmanagedAnsibleAIX" >> inventory.ini
```

Run the first Ansible command to check the SSH connection.

```ansible all -i inventory.ini -l yourmanagedAnsibleAIX -m ping```

Create a simple playbook to check the connection and rights on the Ansible managed AIX.

```vi playbook.yml```

Add the following code to the playbook.yml.

```yaml
- hosts: all
  gather_facts: no
  become: no
  tasks:
    - name: "Ping-check to validate network connection."
      ansible.builtin.ping:

    - name: "Gather data on the managed system, because we set gather_facts to no."
      ansible.builtin.setup:

    - name: "Show debug message with data about the connection."
      ansible.builtin.debug:
        msg: "Hello {{ ansible_user_id }}, you are connected to {{ ansible_hostname }} which is running {{ ansible_os_family }}."

    - name: "Check for superuser"
      block:
        - name: "Execute >whoami< on CLI as superuser."
          ansible.builtin.shell: whoami
          become: yes
          register: superuser_check_result
  
        - name: "Show debug message, if superuser check succeded."
          ansible.builtin.debug:
            msg: "Hello {{ ansible_user_id }}, you were able to run a command as {{ superuser_check_result.stdout }}"

      rescue:
        - name: "Show debug message, if superuser check failed."
          ansible.builtin.debug:
            msg: "Hello {{ ansible_user_id }}, you were not able to switch to superuser."
```

Run the playbook on yourmanagedAnsibleAIX.

```ansible-playbook playbook.yml -i inventory.ini -l yourmanagedAnsibleAIX```

## Setup sudo to allow the Ansible user to execute commands as superuser

Add sudoers rule if you want to allow the Ansible connect user to run commands as superuser.

```echo "youransibleconnectuser ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers```

Re-run the playbook on yourmanagedAnsibleAIX to check if superuser commands work now.

```ansible-playbook playbook.yml -i inventory.ini -l yourmanagedAnsibleAIX```

## Feedback, Issues and Pull-Requests

Feel free to report issues, fork this project and submit pull requests.
