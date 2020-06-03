---
layout: post
title: "UFW for dynamic domains"
description: "How to open a port when your domain changes IP"
---

I have a server online that I only ever access from home, and I use UFW to
firewall it. It's a little tricky to firewall it though, because my IP changes
fairly regularly. I've set up dynamic DNS at home, but it's not possible to set
a firewall rule based on the domain, only based on an IP address.

This script runs on a regular cron-job on my server, and solves the issue
nicely:


```
# fix_firewall.py
import subprocess
import sys


def run_command(command):
    command = command.split()
    return subprocess.run(command, check=True, stdout=subprocess.PIPE).stdout


def get_remote_ip_address(domain):
    stdout = run_command(f'host {domain}')
    return stdout.strip().split()[-1].decode()


def get_firewalled_ip_and_rule_number(port):
    stdout = run_command('ufw status numbered')
    for line in stdout.decode().splitlines():
        if f'{port}/tcp' in line:
            number, *_, ip = line.rsplit(maxsplit=4)
            return number.strip('[] '), ip


def remove_old_rule(rule_number):
    run_command(f'ufw --force delete {rule_number}')


def add_new_rule(ip, port):
    run_command(f'ufw allow from {ip} proto tcp to any port {port}')


if __name__ == '__main__':
    _, domain, port = sys.argv
    remote_ip = get_remote_ip_address(domain)
    rule_number, firewalled_ip = get_firewalled_ip_and_rule_number(port)
    if remote_ip != firewalled_ip:
        print(f'Changing IP from {firewalled_ip} to {remote_ip}.')
        remove_old_rule(rule_number)
        add_new_rule(remote_ip, port)
```

This script has to be run as `root` so that it can modify the firewall rules.

It's run like this:

```
python3 fix_firewall.py my-domain.example.com 12345
```
