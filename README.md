# controls

Ansible role that enables remote control of systems. Example controls include:

- changing the "night light" temperature of all computers at once
- locking/unlocking the screen
- changing the powersave / performance mode (via cpu governor)
- changing the idle monitor poweroff time (via `dpms`)

...and others. The idea is to integrate seamlessly with Ansible and Home
Assistant.

## Requirements

There are no requirements for this, it should work out of the box with Ansible.

Currently, this generally supports only x11 hosts.

## Role Variables

A single Ansible host executes control commands on all of the eligible hosts. A
host is eligible if it has one or more of the appropriately defined variables
set to `true`.

The following variables are used to dictate which controls get applied to hosts:

- `controls_lockable` - allows the screen to be locked/unlocked.
  - `controls_lock_with_slock` can be set to `true` if you prefer to use `slock`
    instead of `loginctl` for lock/unlock, but bear in mind that this will
    require root access and relies on identifying the Xauthority for the current
    display manager (sddm/ly/gdm/etc)
- `controls_redshift` - allows the screen's "night light" temperature to be
  controlled via `redshift`.
- `controls_power` - allows the CPU governor to be changed from values such
  as `balanced`, `powersave`, and `performance`.
- `controls_dpms` - configures the monitor to stay on
  forever (`caffeine` tag), or to turn off after a few seconds of idle time
  (`idle` tag). This is generally used in combination with screen locking to
  ensure that the monitor turns off quickly after locking the screen.

### Additional variables

- `controls_xdisplay` - defaults to `:0`, this is the x display to use for
  control commands

### Suspend and unsuspend

Additionally, the more complex suspend/unsuspend capability is offered as well,
but `etherwake` must be installed on the system that is running Ansible, and
wake-on-lan capabilities must be supported by your network interface(s). You
must set the following variables for hosts that want to be suspended and
unsuspended:

- `controls_suspend` - allows the system to be suspended and unsuspended.
- `controls_wol_nic` - such as `enp1s0`
- `controls_wol_mac` - just set a mac address for this

### Screenshare and unscreenshare

The screenshare/unscreenshare control naively assumes that a user-scoped systemd
service is running called `x11vnc.service`. If you do not have this running, the
control will fail.

- `controls_x11vnc` - must be set to `true` in order for screenshare/unscreenshare
  to be controlled.

## Dependencies

None at the moment.

## Usage

First, create a `requirements.yml` in your Ansible setup if you haven't already,
here's an example:

```yaml
---
collections:
  - name: community.general
  - name: ansible.posix
  - name: community.crypto

roles:
  - name: charles-m-knox.controls
    src: https://github.com/charles-m-knox/ansible-role-controls.git
    scm: git
    version: main
```

Next, install it:

```bash
# include the -f flag to forceably re-install and get the latest (equivalent to
# upgrading)
ansible-galaxy role install -f -r requirements.yml
```

Now, in your `site.yml` (or whatever your playbook is named), use the role -
note that root access is required for some of the steps:

```yaml
- name: controls
  hosts: all
  roles:
    - charles-m-knox.controls
```

Practical usage involves combining a few things at once:

```bash
# locks your screens, makes the monitors turn off after a few seconds, and
# sets cpu to powersaver mode
ansible-playbook site.yml --tags idle,powersaver,lock
# unlocks your screens, makes the monitors stay on, sets cpu to performance mode
ansible-playbook site.yml --tags performance,unlock,caffeine,2200K
```

## License

MIT

## Author Information

<https://charlesmknox.com>
