# Ansible set up Local
1. master - Has ansible
Create it by navigating to terraform/master and giving `terraform apply`
2. test - test server
Create it by navigating to terraform/test and giving `terraform apply`
3. Generate ssh-key in master and add id_rsa.pub key's contents to ~/.ssh/authenticated_keys in test instance
4. Make an ssh connection to test from master and check if it works
5. In the master instance, give the following command
    `mkdir ansible`
    `cd ansible/`
6. Create a file
    ```
    ubuntu@master-instance:~/ansible$ vi hosts
    web ansible_ssh_host=10.128.64.153
    ```
7. Create a file
    ```
    ubuntu@master-instance:~/ansible$ vi ansible.cfg
    [defaults]
    hostfile = hosts
    transport=paramiko
    remote_user = ubuntu
    private_key_file = ~/.ssh/id_rsa
    host_key_checking = False
    ```
8.  Some commands
```
 ubuntu@master-instance:~/ansible$ ansible web -a uptime -k
 SSH password:
 web | SUCCESS | rc=0 >>
 01:45:10 up  4:15,  2 users,  load average: 0.00, 0.01, 0.05
```
```
 ubuntu@master-instance:~/ansible$ ansible web -a "tail /var/log/dmesg" -k
SSH password:
web | SUCCESS | rc=0 >>
[   19.330981] EXT4-fs (vda1): resizing filesystem from 360448 to 2621184 blocks
[   19.403415] EXT4-fs (vda1): resized filesystem to 2621184
```
```
ubuntu@master-instance:~/ansible$ ansible web -s -k -m apt -a name=nginx
SSH password:
web | SUCCESS => {
    "cache_update_time": 0,
    "cache_updated": false,
    ...
    "Setting up nginx (1.4.6-1ubuntu3.5) ...",
        "Processing triggers for libc-bin (2.19-0ubuntu6.9) ..."
    ]
}
```
```
ubuntu@master-instance:~/ansible$ ansible web -s -k -a "ls /etc/nginx"
SSH password:
web | SUCCESS | rc=0 >>
conf.d
fastcgi_param
```
```
ubuntu@master-instance:~/ansible$ ansible web -s -k -m service -a "name=nginx state=restarted"
SSH password:
web | SUCCESS => {
    "changed": true,
    "name": "nginx",
    "state": "started"
}
```
```
ubuntu@master-instance:~/ansible$ ansible-console -k -b
SSH password:
Welcome to the ansible console.
Type help or ? to list commands.
```
```
ubuntu@all (1)[f:5]# id
web | SUCCESS | rc=0 >>
uid=0(root) gid=0(root) groups=0(root)

```

# Run Hellonode application
Run the following command from master instance

```
ubuntu@master-instance:~/ansible$ ansible-playbook node-app.yml -k -v
Using /home/ubuntu/ansible/ansible.cfg as config file
SSH password:
[DEPRECATION WARNING]: Instead of sudo/sudo_user, use become/become_user and make sure become_method is 'sudo' (default).
This feature will be removed in a future release. Deprecation
warnings can be disabled by setting deprecation_warnings=False in ansible.cfg.

PLAY [web] *********************************************************************

TASK [setup] *******************************************************************
ok: [web]

TASK [Install Package] *********************************************************
ok: [web] => (item=[u'build-essential', u'npm', u'nodejs-legacy', u'git', u'mcrypt', u'nginx', u'curl']) => {"cache_update_time": 1475107553, "cache_updated": true, "changed": false, "item": ["build-essential", "npm", "nodejs-legacy", "git", "mcrypt", "nginx", "curl"]}

TASK [Install pm2] *************************************************************
ok: [web] => {"changed": false}

TASK [Create app directory] ****************************************************
ok: [web] => {"changed": false, "gid": 0, "group": "root", "mode": "0755", "owner": "root", "path": "/home/ubuntu/hellonode", "size": 4096, "state": "directory", "uid": 0}

TASK [Generate rsa key pair locally without passphrase. Add its public key to GIT repository. Below is the instruction to copy private key to the target system.] ***
ok: [web] => {"changed": false, "checksum": "2776e56eab60f5c2ceb8ca7dcb4f81fe40790f01", "dest": "/home/ubuntu/.ssh/id_rsa", "gid": 0, "group": "root", "mode": "0600", "owner": "root", "path": "/home/ubuntu/.ssh/id_rsa", "size": 1679, "state": "file", "uid": 0}

TASK [Clone Git Repository] ****************************************************
ok: [web] => {"after": "6c77d406980c36e3c4da0e8845b0fdc30573b9dd", "before": "6c77d406980c36e3c4da0e8845b0fdc30573b9dd", "changed": false, "warnings": []}

TASK [Install dependencies] ****************************************************
skipping: [web] => {"changed": false, "skip_reason": "Conditional check failed", "skipped": true}

TASK [Stop app] ****************************************************************
changed: [web] => {"changed": true, "cmd": ["pm2", "stop", "app"], "delta": "0:00:00.455052", "end": "2016-09-29 00:06:22.407981", "rc": 0, "start": "2016-09-29 00:06:21.952929", "stderr": "", "stdout": "[PM2] Applying action stopProcessId on app [app](ids: 1)\n[PM2] [app](1) ✓\n┌───────────┬────┬──────┬─────┬─────────┬─────────┬────────┬─────┬────────┬──────────┐\n│ App name  │ id │ mode │ pid │ status  │ restart │ uptime │ cpu │ mem    │ watching │\n├───────────┼────┼──────┼─────┼─────────┼─────────┼────────┼─────┼────────┼──────────┤\n│ app       │ 1  │ fork │ 0   │ stopped │ 0       │ 0      │ 0%  │ 0 B    │ disabled │\n│ hellonode │ 0  │ fork │ 0   │ stopped │ 0       │ 0      │ 0%  │ 0 B    │ disabled │\n└───────────┴────┴──────┴─────┴─────────┴─────────┴────────┴─────┴────────┴──────────┘\n Use `pm2 show <id|name>` to get more details about an app", "stdout_lines": ["[PM2] Applying action stopProcessId on app [app](ids: 1)", "[PM2] [app](1) ✓", "┌───────────┬────┬──────┬─────┬─────────┬─────────┬────────┬─────┬────────┬──────────┐", "│ App name  │ id │ mode │ pid │ status  │ restart │ uptime │ cpu │ mem    │ watching │", "├───────────┼────┼──────┼─────┼─────────┼─────────┼────────┼─────┼────────┼──────────┤", "│ app       │ 1  │ fork │ 0   │ stopped │ 0       │ 0      │ 0%  │ 0 B    │ disabled │", "│ hellonode │ 0  │ fork │ 0   │ stopped │ 0       │ 0      │ 0%  │ 0 B    │ disabled │", "└───────────┴────┴──────┴─────┴─────────┴─────────┴────────┴─────┴────────┴──────────┘", " Use `pm2 show <id|name>` to get more details about an app"], "warnings": []}

TASK [Start app] ***************************************************************
changed: [web] => {"changed": true, "cmd": ["pm2", "start", "server.js", "--name", "app"], "delta": "0:00:00.379377", "end": "2016-09-29 00:06:23.535381", "rc": 0, "start": "2016-09-29 00:06:23.156004", "stderr": "", "stdout": "[PM2] Applying action restartProcessId on app [app](ids: 1)\n[PM2] [app](1) ✓\n[PM2] Process successfully started\n┌───────────┬────┬──────┬───────┬─────────┬─────────┬────────┬─────┬──────────┬──────────┐\n│ App name  │ id │ mode │ pid   │ status  │ restart │ uptime │ cpu │ mem      │ watching │\n├───────────┼────┼──────┼───────┼─────────┼─────────┼────────┼─────┼──────────┼──────────┤\n│ app       │ 1  │ fork │ 24658 │ online  │ 0       │ 0s     │ 0%  │ 4.5 MB   │ disabled │\n│ hellonode │ 0  │ fork │ 0     │ stopped │ 0       │ 0      │ 0%  │ 0 B      │ disabled │\n└───────────┴────┴──────┴───────┴─────────┴─────────┴────────┴─────┴──────────┴──────────┘\n Use `pm2 show <id|name>` to get more details about an app", "stdout_lines": ["[PM2] Applying action restartProcessId on app [app](ids: 1)", "[PM2] [app](1) ✓", "[PM2] Process successfully started", "┌───────────┬────┬──────┬───────┬─────────┬─────────┬────────┬─────┬──────────┬──────────┐", "│ App name  │ id │ mode │ pid   │ status  │ restart │ uptime │ cpu │ mem      │ watching │", "├───────────┼────┼──────┼───────┼─────────┼─────────┼────────┼─────┼──────────┼──────────┤", "│ app       │ 1  │ fork │ 24658 │ online  │ 0       │ 0s     │ 0%  │ 4.5 MB   │ disabled │", "│ hellonode │ 0  │ fork │ 0     │ stopped │ 0       │ 0      │ 0%  │ 0 B      │ disabled │", "└───────────┴────┴──────┴───────┴─────────┴─────────┴────────┴─────┴──────────┴──────────┘", " Use `pm2 show <id|name>` to get more details about an app"], "warnings": []}

PLAY RECAP *********************************************************************
web                        : ok=8    changed=2    unreachable=0    failed=0

```
Now the applciation is accessible via test instance using `10.128.64.153:9091`