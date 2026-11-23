
[Ubuntu Server 24.04 Installation video](https://www.youtube.com/watch?v=HZP8C-CeSpQ)

> **Ubuntu Version:**  [Ubuntu Server 24.04.2 LTS](https://releases.ubuntu.com/24.04.2/ubuntu-24.04.2-live-server-amd64.iso)

---
---
### **First switch to root user:**

```bash
sudo su - root
```

> Enter the password of the first user you created during installation(user with sudo privilege).

---
---
### Modify commands' history:

> **Source**: [digitalocean](https://www.digitalocean.com/community/tutorials/how-to-use-bash-history-commands-and-expansions-on-a-linux-vps)

> Add date & time to commands. 

> 10000 Commands max.

> Write each command to the history file immediately, then reload so all sessions share a live, tamper-resistant log.

> Applied for remote and console logins.

**System Wide:**
```bash
# 1. Create the profile.d script — sourced for ALL login shells (console + SSH)
sudo tee /etc/profile.d/bash-history-audit.sh > /dev/null << 'EOF'
# System-wide bash history configuration (console + SSH logins)

# Timestamp every history entry
export HISTTIMEFORMAT="%d/%m/%y %T "

# History size (in-memory / on-disk)
export HISTSIZE=10000
export HISTFILESIZE=20000

# Don't overwrite history file, append to it
shopt -s histappend 2>/dev/null

# Write each command to the history file immediately, then reload
# so all sessions share a live, tamper-resistant log
export PROMPT_COMMAND="history -a; history -n; ${PROMPT_COMMAND}"
EOF

# 2. Make it readable/executable by everyone
sudo chmod 644 /etc/profile.d/bash-history-audit.sh

# 3. Also cover non-login interactive shells (e.g. `su username` without `-`)
#    by having bash.bashrc pull in the same settings
sudo tee -a /etc/bash.bashrc > /dev/null << 'EOF'

# Pull in system-wide history audit settings for non-login interactive shells too
[ -f /etc/profile.d/bash-history-audit.sh ] && . /etc/profile.d/bash-history-audit.sh
EOF
```

> `histappend` → prevents one session from wiping another's history.

> `history -a` → append new commands to `~/.bash_history` **immediately**.

> `history -n` → reload history that other sessions wrote.

---
---
### Install security packages automatically:

```bash
apt install unattended-upgrades
```

---
### Update/upgrade the machine:

```bash
apt update && apt upgrade -y
```

**Get OS Version:** (Optional)
```bash
cat /etc/os-release
```

---
---
### Add `ll` command alias system wide(if it doesn't exist):

**Open the `/etc/bash.bashrc` file:**
```bash
vim /etc/bash.bashrc
```

> **Hint**: Press `shift` + `g` to go to the bottom, then press `o` to insert in a new line.

> If it does't seem to work for any reason add it to `~/.bashrc` of your current user.

**Add the following to the end:**
```bash
# some more ls aliases
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
```

**Make it take effect:**
```bash
source /etc/bash.bashrc
```

> Your can test that it works by logging in remotely(ssh) and using console, the issue the command `ll`.

---
---
### Install Helpful Tools:

```bash
sudo apt install htop btop libpam-google-authenticator net-tools telnet network-manager vim iperf3 curl wget nano chrony iftop smartmontools ncdu tuned pidstat screen sysstat -y
```

> `htop`, `btop`: better hardware utilization visualization.

> `libpam-google-authenticator` for 2 factor authentication password and time one time password `TOTP`.

> `telnet`: test connection to open ports on other machines.

> `network-manager`: can be used with netplan throw GUI:`nmtui` or CLI:`nmcli` commands.

> `net-tools`: for `netstat` command.

> `iperf3`: test network speed.

> `iftop`: see network traffic in and out.

> `smartmontools`: a set of utilities for monitoring the health of storage drives.

> `lm-sensors`: the main command is `sensors`, reads hardware monitoring data from your system: CPU and motherboard temperatures, fan speeds, and voltages.

> `ncdu`:  interactive, text-based CLI utility used to analyze disk space on Unix and Linux systems.

---

**If you are using KVM:**
```bash
apt install qemu-guest-agent -y
systemctl enable qemu-guest-agent
systemctl start qemu-guest-agent
systemctl status qemu-guest-agent
```

> Make sure its enabled on the hypervisor **EXP**: oVirt, Proxmox.

---
---
### Change Host Name:

```bash
hostnamectl set-hostname <NEW_NAME>
```

> Replace <NEW_NAME> 

> Recommended: Use the domain name like `my-website.com` as the hostname.

**To see effect now run:**
```bash
bash
```

**Before:**
```bash
root@ubuntu:~#
```

**After:**
```bash
root@my-website:~#
```

---
---
### Assign A Static IP(Optional):

**Default:** automatic IP .
```bash
network:
  version: 2
  ethernets:
    enp0s3: # Interface Name
      dhcp4: yes # Automatic
```

---

**For ubuntu versions < `24.04` disable `cloud-init`:** to stop it from overwriting netplan.
```bash
vim /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg
```

**Add the following:**
```bash
network: {config: disabled}
```

**Set your static IP:**
```bash
vim /etc/netplan/50-cloud-init.yaml
```

> Default file is usually: `/etc/netplan/50-cloud-init.yaml`.

**Static IP:**
```bash
network:
  version: 2
  ethernets:
    enp0s3: # Interface Name
      dhcp4: no # Static
      addresses: [192.168.1.100/24] # IP/subnet
      routes:
      - to: "default"
        via: "192.168.1.1" # Gateway
      nameservers:
        addresses: [8.8.8.8, 1.1.1.1] # DNS
```

> Change the `IP/subnet`, `Gateway` and `DNS`.

**Apply:**
```bash
netplan apply
```

**Show info:**
```
netplan status enp0s3
```

```output
     Online state: online
    DNS Addresses: 127.0.0.53 (stub)
       DNS Search: .

●  2: enp0s3 ethernet UP (networkd: enp0s3)
      MAC Address: 08:00:27:10:ae:62 (Intel Corporation)
        Addresses: 192.168.1.100/24
                   fe80::a00:27ff:fe10:ae62/64 (link)
    DNS Addresses: 8.8.8.8,1.1.1.1
           Routes: default via 192.168.1.1 (static)
                   192.168.1.0/24 from 192.168.1.100 (link)
                   fe80::/64 metric 256
```

---
---
### Add Custom NTP Server:(Optional)

> If you have internal NTP server.

> NTP Port: 123

> **Note**: Make sure the NTP machine firewall accepts from your machine.

**Set timezone manually:**
```bash
timedatectl set-timezone Africa/Cairo
```

> Change `Africa/Cairo` with your timezone.

**Install `chrony`:**
```bash
apt -y install chrony
```

**Add your server to `/etc/chrony.conf` and remove defaults:**
```bash
vim /etc/chrony/chrony.conf
```

**Comment the following:**
```bash
pool ntp.ubuntu.com        iburst maxsources 4
pool 0.ubuntu.pool.ntp.org iburst maxsources 1
pool 1.ubuntu.pool.ntp.org iburst maxsources 1
pool 2.ubuntu.pool.ntp.org iburst maxsources 2
```

**Add your custom NTP server:**
```text
server <CUSTOM_NTP_IP> iburst
```

**Final result:**
```bash

server <CUSTOM_NTP_IP> iburst

# pool ntp.ubuntu.com        iburst maxsources 4
# pool 0.ubuntu.pool.ntp.org iburst maxsources 1
# pool 1.ubuntu.pool.ntp.org iburst maxsources 1
# pool 2.ubuntu.pool.ntp.org iburst maxsources 2
```

 **Now run:**
```bash
systemctl enable chrony
systemctl restart chrony
systemctl status chrony
```

**Show NTP sources:**
```bash
chronyc sources -v
```

**Output:**
```bash
  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample               
===============================================================================
^* my-custom-ntp.com              10  10   377  1009   +296us[ +495us] +/-  493us
```

**Focus on this part:**
```bash
===============================================================================
^* my-custom-ntp.com              10  10   377  1009   +296us[ +495us] +/-  493us
```

> It Should look something like this.

> `^*`: means `my-custom-ntp.com` NTP server is currently in use, otherwise its not reachable or not in use.

**Show when it will change summer and winter time:(Optional)**
```bash
zdump -v /etc/localtime | grep <YEAR>
```

> Replace `<YEAR>` with anything like: 2025, 2026, 2030.

**Output:** should look something like this.
```bash
/etc/localtime  Thu Apr 23 21:59:59 2026 UT = Thu Apr 23 23:59:59 2026 EET isdst=0 gmtoff=7200
/etc/localtime  Thu Apr 23 22:00:00 2026 UT = Fri Apr 24 01:00:00 2026 EEST isdst=1 gmtoff=10800
/etc/localtime  Thu Oct 29 20:59:59 2026 UT = Thu Oct 29 23:59:59 2026 EEST isdst=1 gmtoff=10800
/etc/localtime  Thu Oct 29 21:00:00 2026 UT = Thu Oct 29 23:00:00 2026 EET isdst=0 gmtoff=7200
```

**Check time and date:**
```bash
timedatectl
```

**Output:** should look something like this.
```bash
               Local time: Fri 2026-10-02 00:00:00 EEST
           Universal time: Thu 2026-10-01 21:00:00 UTC
                 RTC time: Thu 2026-10-01 21:00:00
                Time zone: Africa/Cairo (EEST, +0300)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
```

---
---
### Security and SSH hardening:

#### SSH hardening:

> Check the ssh-hardening part for more details.

> Most Important steps:
> - Disable password login and enable key authentication.
> - Stop root from remotly logging in.
> - Install `fail2ban` for SSH.

##### Lets's just cover basics for now:

 **Disable root login:**
```
PermitRootLogin no
```
 
**Disable empty passwords:**
```
PermitEmptyPasswords no
```

**Change the default port:**
```
Port <NUMBER>
```

> Reduces automated port scanning.

> Replace `<NUMBER>` with any unused port above 1024.

**Limit login attempts and timeout per session:**
```
MaxAuthTries 3
LoginGraceTime 30
```

---
#### Edit the machine's firewall to restrict access:

> By default `ufw` is inactive(disabled).

**Edit defaults and enable the firewall:**
```bash
ufw default deny incoming
ufw default allow outgoing
ufw limit <SSH_PORT>/tcp comment "SSH PORT Limited."
ufw enable
```

> Change `<SSH_PORT>`.

---
##### If your IP is static:

**Allow access to the ssh port from static IP:**
```bash
ufw allow from <YOUR_IP> to any port <SSH_PORT> proto tcp comment "<COMMENT_HERE>"
```

> Change `<SSH_PORT>` and `<YOUR_IP>`.
##### If your IP is dynamic:

> To reduce the risk of brute-force login attempts, you can rate-limit SSH instead of simply allowing it. The `limit` command caps new connections at `6` attempts within `30` seconds from any single IP address.

> Use `fail2ban` to jail abusing IPs.

**Allow access to the new ssh port from anywhere:**
```bash
ufw limit <SSH_PORT>/tcp comment "<COMMENT_HERE>"
```

> Change `<SSH_PORT>`.

---

**Show firewall rules:**
```bash
ufw status verbose
```

**Show firewall rules numbered:**
```bash
ufw status numbered
```

**Delete a rule:**
```bash
ufw delete <NUMBER>
```

---

**Test if the new port works from your PC:**
```bash
ssh -p <NEW_SSH_PORT> <USER>@<MACHINE_IP>
```

> Use your favorite method to ssh into your machine like `mobaxterm`.

---
#### Install And Config `google-authenticator`: (Extra Security)

> Two-factor authentication (2FA).

> Here I'm using `password` + `google-authenticator` which is not recommended, use key + `google-authenticator` instead, check the ssh hardening part.

**Run `google-authenticator`:**
```bash
google-authenticator
```

> All answers to questions are (yes).

> Scan the QR-code with the google-authenticator mobile app.

> Copy the `emergency scratch codes` and store them somewhere safe in case you can't use google authenticator or there is an NTP problem.

> If you use one of the `emergency scratch codes` you can add it back at the end of this file: `~/.google_authenticator`.

**Edit The Following Files:**
```bash
vim /etc/pam.d/sshd
```

**Add to the top with same order, password then code:**
```bash
auth requisite pam_unix.so try_first_pass
auth required pam_google_authenticator.so
```

---

**Edit `/etc/ssh/sshd_config`:**
```bash
sudo vim /etc/ssh/sshd_config
```

> **Make sure these two are uncommented and with value `yes`.**

> If you can't find them add them specially `ChallengeResponseAuthentication`.

```bash
PasswordAuthentication yes
ChallengeResponseAuthentication yes
```

---

**Restart ssh service to take effect:**
```bash
systemctl daemon-reload \
&& systemctl restart ssh
```

---
---
### Local Domain With `Avahi`:(Optional)

> To be able to reference to the machine with the local hostname like `database.local`, `myshop.local`, `ubuntu.local`.

> Default port: 5353/udp.

> Don't forget to open the port in the firewall, but restrict it to whomever needs it.

```bash
apt install avahi-daemon -y
```

**Check status:**
```bash
systemctl status avahi-daemon.service
```

**Enable and start service:** If needed.
```bash
systemctl enable --now avahi-daemon.service
```

---