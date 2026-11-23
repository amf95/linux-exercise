[Ubuntu Server 24.04 Installation Video](https://www.youtube.com/watch?v=HZP8C-CeSpQ)

> **Ubuntu Version:**  [Ubuntu Server 24.04.2 LTS](https://releases.ubuntu.com/24.04.2/ubuntu-24.04.2-live-server-amd64.iso)

---

> **Note:** This tutorial assumes that the machine has internet access.

> **Note:** Moving around is with keyboard `arrows`, choosing something is with `Enter` of `Space`.

---

**Choose `Try or Install Ubuntu Server`:**

> Press `Enter` to get in faster.

![](assets/0-installation-boot.png)

---

**Choose `English`:**

![](assets/1-choose-english.png)

---

**Choose `Done`:**

![](assets/2-choose-done.png)

---

**Select `Ubuntu Server` and `Search for third-party drivers` then `done`:**

![](assets/3-check-ubuntu-server-and-seach-for-third-party-drivers-then-done.png)

---

**(Option_01) Dynamic IP:**

> `DHCP` will give you an automatic IP.

> You can set static IP in (Option_02).

**Choose `Done`:**

![](assets/4-choose-done.png)

--- **or** ---

**(Option_02) Static Manual IP:**

| Subnet Mask Format | CIDR (Slash / Prefix) Notation | Dotted-Decimal Notation           |
| ------------------ | ------------------------------ | --------------------------------- |
| **EXP**            | 22, 24, ...                    | 255.255.252.0, 255.255.255.0, ... |

| Attribute   | Value                                                                       | EXP             |
| ----------- | --------------------------------------------------------------------------- | --------------- |
| Subnet      | <NETWORK_ID>/<SUBNET_MASK>                                                  | 192.168.1.0/24  |
| Address     | <MACHINE_STATIC_IP>                                                         | 192.168.1.111   |
| Gateway     | Usually the fist usable IP in your network or VLAN                          | 192.168.1.1     |
| Name server | Use your local DNS if you have one, otherwise use google's or clouldflare's | 8.8.8.8,1.1.1.1 |


![](assets/Pasted%20image%2020260911110500.png)

**Choose `Save`:**

---

**Choose `Done`:**

> We don't have a proxy so skip by leaving empty then choose `Done`:

![](assets/5-choose-done.png)

---

**Choose `Done`:**

> Unless you have a specific mirror leave as is and choose `Done`.

![](assets/6-choose-done.png)

---

**Choose `Done`:** 

> Make sure `Set up this disk as in LVM group` is selected(default).

> Unless you want a custom partitioning select `Done`.

> **Note**: by default not all of the available space is occupied by the LVM disk but you can expand it later or during installation.

![](assets/7-choose-done.png)

---

**(Option_1) Choose `Done`:** If you are gonna expand the `/` later.

![](assets/8-choose-done.png)

--- **or** ---

**(Option_2) Expand `/` to use all available space:**

**Unmount `/`:**

![](assets/unmount-root-mount-point.png)

**Select `ubuntu-lv` -> Press `Enter` -> Select `Edit` -> Press `Enter`:**

![](assets/select-ubuntu-lvm-edit.png)

**Select `Size` then type desired size:** Max size in this case.

> **Note:** If you type a bigger value than available space it will still give it max available space.

![](assets/type-max-value.png)

**Select Mount -> Press `Enter` -> Select `/` -> Press `Enter`:**

![](assets/select-root.png)

**Select `Save` -> Press `Enter`:**

![](assets/select-save.png)


---

**Choose `Continue`:**

![](assets/9-choose-continue.png)

---

**Enter your `username`, `servername` <-> `hostname` and `password` then `Done`:**

![](assets/10-enter-credentials-then-done.png)

---

**Select `Skip for now` then `Continue`:**

![](assets/11-choose-skip-then-continue.png)

---

**Select `Install OpenSSH server` then `Done`:**

![](assets/12-choose-install-openssh-server-then-done.png)

---

**Choose `Continue`:**

> No drivers to install.

![](assets/13-choose-continue.png)

---

**Choose `Done`:**

> Select the package you want to install if you want.

![](assets/14-choose-done.png)

---

**Wait till the `Orange Bar`  at top says `Installation Complete!`:**

![](assets/15-installing.png)
![](assets/16-installion-complete.png)

> **Note:** Now you have `Reboot Now` option.

---

**Choose `Reboot Now`:**

![](assets/17-choose-reboot-now.png)

---

**Press `Enter`:**

> Don't worry this is not an error.

> Don't Forget To Remove The CD Installation Media After Your Are Done, in case its not removed automatically by the hypervisors.

![](assets/18-press-enter.png)

---

**Enter you `username` then `password`:**

> You set during the installation process.

![](assets/19-enter-your-user-and-password.png)

---

**Login Success:**

![](assets/20-login-success.png)

