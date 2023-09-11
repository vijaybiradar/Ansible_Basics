

![image](https://github.com/vijaybiradar/Ansible_Basics/assets/38376802/8c05814f-b9ea-4936-a7d6-41920ba840c4)



playbook for executing the 'date' command on the localhost, here's a more concise version:

```
---
- name: Play 1
  hosts: localhost
  tasks:
    - name: Execute command 'date'
      command: date
```
In this playbook:

The name field defines the name of the playbook.
The hosts field specifies the target hosts, in this case, just the localhost.
The tasks section contains the list of tasks to be executed on the specified host(s).
Inside the tasks, there's a single task named "Execute command 'date'" that runs the 'date' command.
This playbook will execute the 'date' command on the localhost when you run it with Ansible. It's a simple and straightforward playbook for this specific task.


```
---
---
- name: Get the current date and time
  hosts: all
  gather_facts: yes
  become: yes  # Use become to run tasks as root
  become_user: root  # Specify the user as root
  tasks:
    - name: Get the current date
      ansible.builtin.command: date
      async: 3600
      poll: 0
      ignore_errors: yes
      register: date_result
      notify:
        - Handle date command result

  handlers:
    - name: Handle date command result
      debug:
        msg: |
          The 'date' command on {{ inventory_hostname }} executed {{ 'successfully' if date_result.rc == 0 else 'with failure' }}.
          Output: {{ date_result.stdout | default('No output') }}
      loop: "{{ ansible_play_hosts }}"
      when: date_result.rc is defined and date_result.rc != 0

    - name: Notify on failure
      debug:
        msg: |
          The 'date' command on {{ inventory_hostname }} failed with error: {{ date_result.stderr | default('No error message') }}
      loop: "{{ ansible_play_hosts }}"
      when: date_result.rc is defined and date_result.rc != 0

```
Now, let's break down and explain each component in detail:

1. Playbook Header:

```
- name: Get the current date and time
  hosts: all
  gather_facts: yes
  become: yes  # Use become to run tasks as root
  become_user: root  # Specify the user as root
```
name: This is the name of the playbook, which is "Get the current date and time." It provides a description of the playbook's purpose.

hosts: Specifies the target hosts where the tasks will be executed. In this case, it's set to all, meaning it will run on all hosts defined in your Ansible inventory file.

gather_facts: When set to yes, Ansible gathers system facts from the target hosts before executing tasks. Gathering facts provides valuable information about the target systems and can be useful for task execution. It's set to yes to ensure facts are collected.

become: yes: The become directive is added at the playbook level, and it's set to yes to enable privilege escalation. This allows the tasks in this playbook to be executed with elevated privileges.

become_user: root: The become_user directive specifies the user as root. This means that the tasks will be executed as the root user after privilege escalation.

2. Tasks:

```
  tasks:
    - name: Get the current date
      ansible.builtin.command: date
      async: 3600
      poll: 0
      ignore_errors: yes
      register: date_result
      notify:
        - Handle date command result
```
name: This is the name of the task, which is "Get the current date." It describes the purpose of the task, which is to run the 'date' command.

ansible.builtin.command: The ansible.builtin.command module is used to execute the 'date' command on the target hosts.

async: This parameter makes the task run asynchronously for up to 3600 seconds (1 hour). Asynchronous execution allows Ansible to continue with other tasks without waiting for this one to finish.

poll: It is set to 0, meaning Ansible will immediately check the status of the asynchronous task. This is useful for determining when the task completes.

ignore_errors: When set to yes, Ansible continues executing other tasks even if this task fails. This is useful if you want to capture the result but not stop the playbook on errors.

register: The register keyword allows you to capture the result of the task in a variable named date_result. This variable will hold information about the command's exit code (rc), standard output (stdout), and standard error (stderr).

notify: The notify directive is used to trigger a handler named "Handle date command result" when this task completes.

3. Handlers:

```
  handlers:
    - name: Handle date command result
      debug:
        msg: |
          The 'date' command on {{ inventory_hostname }} executed {{ 'successfully' if date_result.rc == 0 else 'with failure' }}.
          Output: {{ date_result.stdout | default('No output') }}
      loop: "{{ ansible_play_hosts }}"
      when: date_result.rc is defined and date_result.rc != 0

    - name: Notify on failure
      debug:
        msg: |
          The 'date' command on {{ inventory_hostname }} failed with error: {{ date_result.stderr | default('No error message') }}
      loop: "{{ ansible_play_hosts }}"
      when: date_result.rc is defined and date_result.rc != 0
```
Handlers are defined in a separate section. In this case, there are two handlers: "Handle date command result" and "Notify on failure."

The "Handle date command result" handler prints a message when the 'date' command has a non-zero exit code (rc) indicating failure. It includes information about whether the command executed successfully and its output.

The "Notify on failure" handler prints a message when the 'date' command has a non-zero exit code (rc), indicating failure. It includes information about the error message.

With these additions, the playbook includes handlers to provide detailed output for both success and failure scenarios and uses the notify directive to trigger the appropriate handler. The ignore_errors directive is also used to handle errors gracefully during the execution of the 'date' command. This playbook is comprehensive and provides clear visibility into the task's execution on multiple hosts.



Ansible Basic (Ad-Hoc) Commands

![image](https://github.com/vijaybiradar/Ansible_Basics/assets/38376802/cbddd05e-20b6-4fdd-a596-56a4b2a07398)

Ansible Ad-Hoc commands offer a quick way to automate tasks on remote servers without the need for full Ansible playbooks. They are great for immediate actions and troubleshooting. The basic structure is:

```
ansible <host_or_group> -m <module> -a "<arguments>"
```
Here are essential Ad-Hoc commands:

Reboot a Host:

To reboot a host named "webserver," execute:

```
ansible webserver -m reboot
```
Check Uptime:

To check the uptime of "dbserver," run:

```
ansible dbserver -m command -a "uptime"
```
File Copy:

To copy a file from "webserver" to "dbserver," use:

```
ansible webserver -m copy -a "src=/etc/hosts dest=/tmp/hosts"
```
User Management:

To create a user named "ansible" with a password on all hosts, employ:

```
ansible all -m user -a "name=ansible password=password123"
```
Install Software:

To install a package like "nginx" on all hosts, issue:

```
ansible all -m yum -a "name=nginx state=present"
```
Check Connectivity:

To ping hosts and confirm reachability, use:

```
ansible all -m ping
```
Execute Shell Command:

To run a shell command on a host, execute:

```
ansible webserver -m shell -a "your_shell_command_here"
```
Manage Services:

To start or stop a service like "apache" on a host, employ:

```
ansible webserver -m service -a "name=apache state=started"
```
Manage Files:

To create or delete a file on a host, apply:

```
ansible webserver -m file -a "path=/path/to/file state=touch"
```

