Ansible Atlassian BambooAgent install DMG role
==============================================

Installs the content of a `DMG` file onto a remote target (OSX only).

This is currently a role, but it should rather be a complex command. This role

- copies a DMG file to the host
- transforms it if needed to make automation possible (for instance some DMG require manual inputs: the mentioned transformation remove the manual steps)
- mount it to a temporary folder
- run a command for the installation
- un-mounts and delete any intermediate file (eg. the transformed DMG)

Requirements
------------
No particular requirement apart from running on an OSX machine.

Role Variables
--------------

| variable | default | meaning |
|----------|---------|---------|
|dmg_to_install| **required**| the DMG file to install. This is actually a dictionary indicating
the installation options.|

### ``dmg_to_install`` format
The ``dmg_to_install`` is a dictionary containing the following fields

| field | default | meaning |
|----------|---------|---------|
|``file``| **required**| Points to the DMG file.|
|``install_cmd``| **required**| The command that should be run on the remote after the
DMG has been properly mounted.|
|``remove_interactive``| ``False``| Indicates if the original DMG should be modified in
order to remove some interactive required inputs.|

During the installation, the `${mount}` variable is replaced with the actual mount location of the DMG content (see example below).

Dependencies
------------

No additional dependency.

Example Playbook
----------------

The following example installs the XCode command line tools:

```yaml
# Installs OSX command line tools (as defined in the inventory)
# fixes the paths on the fly
- role: ansible-atlassian-bambooagent-install-dmg-role
  dmg_to_install:
    - "{{ bamboo_xcode }}"
  when: ({{ (ansible_distribution=="MacOSX") and
            (xcode_version_installed | version_compare('%d.%d' % (bamboo_xcode.version.major, bamboo_xcode.version.minor), '<') ) }})
```

and the ``bamboo_xcode`` variable contains the following:

```yaml
bamboo_xcode:
  file: 'commandlinetoolsosx10.10forxcode6.3.2.dmg'
  install_cmd: 'installer -allowUntrusted -dumplog -pkg "${mount}/Command Line Tools (OS X 10.10).pkg" -target /'
  remove_interactive: False
  version:
    major: 6
    minor: 3
    patch: 2
```

The `${mount}` variable is expanded to the actual mount location (random folder).
The `bamboo_xcode.version` is not related to the DMG role directly (but rather on the ansible *condition* of the above `dmg_to_install` command).

License
-------

BSD

Author Information
------------------

Any comments on the Ansible, PR or bug reports are welcome from the corresponding Github project.
