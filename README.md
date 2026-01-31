Ansible role: raspberrypi
=========================

Configures Raspberry Pi system configuration (config.txt) and
bootloader configuration. Updates bootloader EEPROM firmware
and configures a systemd timer to keep firmware up to date.
Configures Raspberry Pi OS to look more like a standard Debian
installation.

Role Variables
--------------

```
ENTRY POINT: *main* - Configure Raspberry Pi devices and Raspberry Pi OS

          Configures Raspberry Pi system configuration (config.txt)
          and bootloader configuration. Updates bootloader EEPROM
          firmware and configures a systemd timer to keep firmware up
          to date. Configures Raspberry Pi OS to look more like a
          standard Debian installation.

Options (= indicates it is required):

- raspberrypi_bootloader_config  Raspberry Pi bootloader
                                  configuration, see
                                  https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#configuration-properties
                                  for more details, or empty string to
                                  leave file as is
          default: ''
          type: str

- raspberrypi_bootloader_config_file  Raspberry Pi bootloader
                                       configuration filename
          default: /boot/firmware/config.txt
          type: str

- raspberrypi_config  Raspberry Pi system configuration file
                       contents, see http://rptl.io/configtxt for more
                       details, or empty string to leave file as is
          default: ''
          type: str

- raspberrypi_config_filename  Raspberry Pi system configuration
                                filename
          default: /boot/firmware/config.txt
          type: str

- raspberrypi_os_issue_file  Filename to use for detecting if host is
                              running Raspberry Pi OS
          default: /etc/rpi-issue
          type: str

- raspberrypi_os_journald_storage  Where to store systemd journal
                                    data, see
                                    https://www.freedesktop.org/software/systemd/man/latest/journald.conf.html#Storage=
          default: volatile
          type: str

- raspberrypi_os_packages_remove  List of Raspberry Pi OS apt
                                   packages to remove
          default: [avahi-daemon, cifs-utils, dnsmasq-base, mkvtoolnix, network-manager, nfs-common,
            ntfs-3g, ppp, raspberrypi-net-mods, triggerhappy, userconf-pi, v4l-utils]
          elements: str
          type: list

- raspberrypi_os_raspi_mirror_components  Components to use for the
                                           Raspberry Pi OS raspi apt
                                           mirror
          default: [main]
          elements: str
          type: list

- raspberrypi_os_raspi_mirror_gpg_key  GPG key to use for the
                                        Raspberry Pi OS raspi apt
                                        mirror
          default: /usr/share/keyrings/raspberrypi-archive-keyring.pgp
          type: str

- raspberrypi_os_raspi_mirror_types  Which types of packages to look
                                      for (deb or deb-src) on the
                                      Raspberry Pi OS raspi apt mirror
          default: [deb]
          elements: str
          type: list

- raspberrypi_os_raspi_mirror_url  Raspberry Pi OS raspi apt mirror
                                    URL
          default: http://archive.raspberrypi.com/debian/
          type: str

- raspberrypi_os_remove_sudo_config  If true, remove Raspberry Pi OS
                                      sudoers configuration files
          default: true
          type: bool

- raspberrypi_os_sudo_config_files  List of Raspberry Pi OS sudoers
                                     configuration files to remove
          default: [010_at-export, 010_dpkg-threads, 010_global-tty, 010_pi-nopasswd, 010_proxy]
          elements: str
          type: list

- raspberrypi_update_eeprom  If true, update Raspberry Pi bootloader
                              EEPROM to latest version
          default: true
          type: bool

- raspberrypi_update_eeprom_randomized_delay  Delay the Raspberry Pi
                                               bootloader EEPROM
                                               update timer by a
                                               random time up to this
                                               value or empty string
                                               for no delay
          default: 6h
          type: str

- raspberrypi_update_eeprom_time  How often to update Raspberry Pi
                                   bootloader EEPROM, accepts a
                                   systemd time, see
                                   https://www.freedesktop.org/software/systemd/man/latest/systemd.time.html,
                                   or "never"
          default: daily
          type: str
```

Installation
------------

This role can either be installed manually with the ansible-galaxy CLI tool:

    ansible-galaxy install git+https://github.com/wandansible/raspberrypi,main,wandansible.raspberrypi

Or, by adding the following to `requirements.yml`:

    - name: wandansible.raspberrypi
      src: https://github.com/wandansible/raspberrypi

Roles listed in `requirements.yml` can be installed with the following ansible-galaxy command:

    ansible-galaxy install -r requirements.yml

Example Playbook
----------------

    - hosts: all
      roles:
         - role: wandansible.raspberrypi
           become: true
           vars:
             raspberrypi_bootloader_config: |
               [all]
               # Enable UART debug output
               BOOT_UART=1

               # Try NVME first, followed by SD card then repeat
               BOOT_ORDER=0xf16

             raspberrypi_config: |
               {% if ansible_distribution == "Ubuntu" %}
               kernel=vmlinuz
               cmdline=cmdline.txt
               initramfs initrd.img followkernel
               {% else %}
               # Automatically load initramfs files, if found
               auto_initramfs=1
               {% endif %}

               # Enable DRM VC4 V3D driver
               dtoverlay=vc4-kms-v3d
               max_framebuffers=2

               # Don't have the firmware create an initial video= setting in cmdline.txt.
               # Use the kernel's default instead.
               disable_fw_kms_setup=1

               # Run in 64-bit mode
               arm_64bit=1

               # Disable compensation for displays with overscan
               disable_overscan=1

               # Run as fast as firmware / board allows
               arm_boost=1

               # Enable UART on GPIOs 14 & 15
               enable_uart=1

               # Enable UART output for second-stage boot loader
               uart_2ndstage=1

               [pi5]
               # Enable UART on GPIOs 14 & 15 and redirect serial0 to there rather than debug connector
               dtparam=uart0_console

               [cm4]
               # Enable host mode on the 2711 built-in XHCI USB controller.
               # This line should be removed if the legacy DWC2 controller is required
               # (e.g. for USB device mode) or if USB support is not required.
               otg_mode=1

               [all]
