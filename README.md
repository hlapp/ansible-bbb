# Ansible playbooks for Beaglebone Black

Ansible playbooks and roles for setting up [Beaglebone Black] boards,
and for deploying the [Ninjablock] software on systems that have
otherwise been properly set up.

## Motivation

This project originally grew out of a desire to have a documented
fully reproducible mechanism for taking a bare-bones (no pun intended)
[Ubuntu image from ELinux] all the way to a fully installed
Ninjablocks device.

The later batches of Ninjablocks (mine is from those delivered 2013,
from the [2012 Ninjablocks Kickstarter]) are essentially Beaglebone
Black boards (with a 2GB eMMC) with an Arduino-based cape. It ran
Ubuntu 13.04, with installation of software controlled by a byzantine
maze of shell scripts. Most software was very outdated, and the
Ninjablocks software itself was not only far from the latest versions
available from the company's Github repositories, but also contained a
good deal of local (uncommitted) changes. All this made it very difficult
or impossible to test upgrades of any of the various components while
retaining the possibility to either re-run or roll back changes.

The ELinux Ubuntu "console" images are quite minimal, and therefore
before Ninjablocks can be deployed on top of them, they must undergo a
variety of networking and other basic system configuration and
software setup steps. This part is entirely independent of
Ninjablocks, and has been separated into its own 'role'.

## How to use

The following playbooks and roles are currently available:

* `bbb.yml`: playbook that applies the `bbb-init` role to hosts in the
  `freshly-imaged` group.

  The playbook can start from a BBB plugged into the USB port of the
  host computer running Ansible and reachable at the default USB
  gadget IP 192.168.7.2. To limit the playbook to this use-case,
  append `--limit bb-usb` to the `ansible-playbook` command line (see
  below).

* `roles/bbb-init`: role that initializes Beaglebone Black boards,
  starting from a (quite minimal) Elinux Ubuntu "console" image. Tasks
  in the role include setting timezone, changing hostname, configuring
  wireless network, ensuring network is up, and end in a reboot. The
  role's play will fail if it fails to either find working network
  (which will typically be via ethernet), or to bring up wireless network.

* `ninjas.yml`: playbook that applies the `ninjablock` role to hosts
  in the `ninjas` group. You can ignore this playbook and the
  `ninjablock` role if you aren't looking to install a Ninjablock.

* `roles/ninjablock`: role that installs all Ninjablocks software and
  dependencies. So that it can be applied independently, the role is
  intentionally not declared dependent on `bbb-init`, although in
  practice almost all tasks in the role will fail if network and other
  system setup tasks have not been performed yet.

The playbooks can be run in the following way:

```sh
$ ansible-playbook -k -K -v <playbook>
```

Local configuration variables can be set (and others overridden) on
the command line using `-e key=value`:

```sh
$ ansible-playbook -k -K -v -e timezone=America/New_York bbb.yml
```

The following variables are by default expected to be set from the
command line; they have no default, and the corresponding system set
up tasks are skipped for those that aren't set:

* `timezone`: name of timezone, if it is to be configured
* `new_hostname`: hostname to change the board to
* `wpa_ssid`: SSID of wireless network to which the board should be
  able to connect. Wireless network configuration is skipped if not set.
* `wpa_passkey`: passkey for the wireless network.
* `wpa_keymgmt`: wireless key management, defaults to `WPA-PSK`.

Typical applications will likely not have a whole farm of
BBBs. Ansible allows limiting the hosts to which a playbook is applied
via the `--limit` parameter. For example, to limit to the default
hostname under which a board with the Ubuntu image would appear on the
local network, you would do this:

```sh
$ ansible-playbook -k -K -v -e timezone=America/New_York bbb.yml --limit arm.local
```

## Why Ansible

Obviously, all the tasks executed by the Ansible playbooks could be
effected by shell scripts as well, and most use-cases (including my
own) will likely not involve a whole complex farm of BBB boards. So
why not just run a bunch of shell scripts on the one or two target
boards?

Based on spending countless hours trying to reverse engineer how a
Ninjablock environment is set up and deployed factory-wise, shell
scripts can be byzantine and very challenging to decipher in terms of
what they do under which conditions with which outcomes. Shell scripts
are also almost always written to assume a certain starting
environment. If this starting point can't be reliably recreated, for
example because the exact starting image isn't available - or, for
that matter, desirable - anymore, then shell scripts are at increased
risk of failing at some step(s) in typically difficult to diagnose and
debug ways.

Though perhaps not as evident but IMO as or even more important,
Ansible playbooks have status-checking built in through their
declarative nature. Most tasks are (or can be) written by declaring
the desired state. If this state is already present, no action is
taken. This makes is possible to run the same playbook whether none,
some, or all of the tasks have already been done previously, without
any alterations such as conditionals etc.

## Prerequisites

Obviously, you must have Ansible installed. A version of at least 1.9+
is required. (At present, the roles on purpose do not use features
available only since Ansible 2.+. This means that for example on Mac
OSX, installing Ansible by `homebrew` will suffice.)

The Beaglebone board is assumed to be a Beaglebone Black; the `bbb`
playbook may also work with other boards in the Beaglebone family, but
this is untested (and I only own BBBs).

At present, the Linux distribution is assumed to be Ubuntu,
release 14.04 ("trusty"). This is in part because Ninjablocks, which
originally motivated this project, use Ubuntu, and in part because (at
least for "console") the Ubuntu images from ELinux require (in my
experience anyway) much more setup to have functioning network and a
reasonable system basis than the Debian images. I intend to expand the
roles to Debian in the future. (One issue here is that Debian Jessie
uses systemd, whereas Ubuntu Trusty still uses Upstart.)

## License

This code is released under the terms of the MIT License, please see
file LICENSE.

[Ninjablock]: https://discuss.ninjablocks.com/category/ninja-block
[Beaglebone Black]: http://beagleboard.org/BLACK
[Ubuntu image from ELinux]: http://elinux.org/BeagleBoardUbuntu#raw_microSD_img
[2012 Ninjablocks Kickstarter]: https://www.kickstarter.com/projects/ninja/ninja-blocks-connect-your-world-with-the-web/
