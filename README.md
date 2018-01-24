## What is ansible-iptables? [![Build Status](https://secure.travis-ci.org/nickjj/ansible-iptables.png)](http://travis-ci.org/nickjj/ansible-iptables)

It is an [Ansible](http://www.ansible.com/home) role to:

- Install iptables
- Configure iptables

## Why would you want to use this role?

Locking down your server with a firewall is an important security step. This
role gives you full control over how you configure iptables.

## Supported platforms

- Ubuntu 16.04 LTS (Xenial)
- Debian 8 (Jessie)
- Debian 9 (Stretch)

## Role variables

```
# Rather than make a big convoluted mess by trying to parse an iptables config
# file, you can just supply a raw config file.
#
# The default set up will lock down your server, and then open up:
# port 22 (SSH) and allow pings from the outside world.
iptables_config: |
  *filter

  # --- Disallow everything as the default filter policy.
  :INPUT DROP [0:0]
  :FORWARD DROP [0:0]
  :OUTPUT ACCEPT [0:0] 

  # --- Allow unrestricted traffic on a few local network adapters.
  -A INPUT -i lo -j ACCEPT
  -A INPUT -i eth1 -j ACCEPT
  -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

  # --- Allow the outside world to connect to SSH (22).
  -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT

  # --- Allow the outside world to send all ICMP requests (ping, etc.).
  -A INPUT -p icmp -m icmp --icmp-type 255 -j ACCEPT

  COMMIT

# A list of 0 or more services to get restart after your iptables rules are
# changed. For instance, Docker makes a number of iptables changes and must be
# loaded after iptables.
#
# All of this is taken care of by this role and my Docker role, and in the case
# where the service (Docker in this case) isn't installed, this task will skip.
#
# Keep in mind, if this does get ran then your service will restart. In the
# case of Docker, all of your containers will restart too, but that's what
# you need to do if you change iptables around. Try not to do it ever / often!
iptables_restart_dependent_services:
  - name: "docker"
    path: "/etc/systemd/system/"
```

## Example usage

For the sake of this example let's assume you have a group called **app** and
you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---

- name: "Configure app server(s)"
  hosts: "app"
  become: True

  roles:
    - { role: "nickjj.iptables", tags: "iptables" }
```

Let's say you wanted to allow HTTP / HTTPS traffic, you can do this by opening
or creating `group_vars/app.yml` which is located relative to your `inventory`
directory and then making it look like this:

```
---

iptables_config: |
  *filter

  # --- Disallow everything as the default filter policy.
  :INPUT DROP [0:0]
  :FORWARD DROP [0:0]
  :OUTPUT ACCEPT [0:0] 

  # --- Allow unrestricted traffic on a few local network adapters.
  -A INPUT -i lo -j ACCEPT
  -A INPUT -i eth1 -j ACCEPT
  -A INPUT -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT

  # --- Allow the outside world to connect to SSH (22), HTTP (80) and HTTPS (443).
  -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
  -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
  -A INPUT -p tcp -m tcp --dport 443 -j ACCEPT

  # --- Allow the outside world to be able to ping you.
  -A INPUT -p icmp -m icmp --icmp-type 0 -j ACCEPT
  -A INPUT -p icmp -m icmp --icmp-type 3 -j ACCEPT
  -A INPUT -p icmp -m icmp --icmp-type 11 -j ACCEPT

  COMMIT
```
Now you would run `ansible-playbook -i inventory/hosts site.yml -t iptables`.

## Installation

`$ ansible-galaxy install nickjj.iptables`

## Ansible Galaxy

You can find it on the official
[Ansible Galaxy](https://galaxy.ansible.com/nickjj/iptables/) if you want to
rate it.

## License

MIT
