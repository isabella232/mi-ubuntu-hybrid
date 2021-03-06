<!--
    This Source Code Form is subject to the terms of the Mozilla Public
    License, v. 2.0. If a copy of the MPL was not distributed with this
    file, You can obtain one at http://mozilla.org/MPL/2.0/.
-->

<!--
    Copyright 2019 Joyent, Inc.
-->

# Ubuntu 16.04+ Image Converter

The machine images created by Canonical need some minor changes to work well
with bhyve. Most importantly, `grub` needs to be configured such that `grub` and
the kernel use the first serial console as the primary console.  Without this
change, it is not feasible to debug or repair bhyve instances from the console.

## Requirements

In order to use this repo, you will need to have the following:

* SmartOS running on a machine that is capable of running bhyve
* An image published by Canonical that needs to be converted.

For the image to support bhyve, it needs to have cloud-init 18.2 or later.
Ubuntu 16.04 and later images created by Canonical after June 2018 have a
version of cloud-init that has the required support even if the version is an
earlier release in the 18.x series.

## Setup

This repo uses `linux-prepare-image` from the
[sdc-imgapi](https://github.com/joyent/sdc-imgapi) repo. sdc-imgapi is a
submodule.

```
$ git submodule update --init
```

## Using

The `create-image` script will create the image.  Under the covers it does the
following:

1. Imports an image provided by Canonical, if necessary
2. Creates a bhyve instance without booting it
3. Creates a new image based on this instance, customizing it with
   `sdc-imgapi/tools/prepare-image/linux-prepare-image`.

### Example: usage

```
# ./create-image -h
Usage:
    create-image -h
    create-image -p
    create-image -u source_image_url [-v version]
    create-image -i source_image [-v version]

Options:
    -d		Debug.  Do not perform cleanup tasks.
    -h		Show this help message
    -i image	Create a new image based on this image that is stored in the
		Joyent manta account of cpcjoyentpublishing.
    -u srcurl	Create an image based on this source image.  The srcurl is
		the url to cloud image tar file distributed by Canonical.
    -v version	Use the supplied version as the minor version.  Defaults to 1.
		If "-v 42" is used and the image provided by Canonical has
		version 20190514, the newly created image will be version
		20190514.42.

The tar file specified with the -i or -u options must be signed with the
"Ubuntu Cloud Image Build (Canonical Internal Cloud Image Builder)
<ubuntu-cloudbuilder-noreply@canonical.com>" gpg key.  The signature must be
found in <tarfile>.gpg.  See https://wiki.ubuntu.com/SecurityTeam/FAQ.
```

### Example: Create image from a locally cached image

The `-u` option may be used to specify any URL that `curl` may use. In this
example, rather than downloading from Manta, we use a copy that is cached in
/var/tmp.

```
# ./create-image -u file:///var/tmp/ubuntu-certified-18.04-20190405.tar -v 0.1
Downloading ubuntu-certified-18.04-20190405
```

### Example: Create image from default incoming location

When Canonical creates new cloud images, they upload them to a public location
in Manta.  Use the `-i` option to convert image found at that location.

```
# ./create-image -i ubuntu-certified-18.04-20190405.tar -v 0.1
Downloading ubuntu-certified-18.04-20190405
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  336M  100  336M    0     0  10.7M      0  0:00:31  0:00:31 --:--:-- 12.8M
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   836  100   836    0     0   1231      0 --:--:-- --:--:-- --:--:--  1416
gpg: WARNING: unsafe ownership on homedir `/zones/root/ubuntu-hybrid/mi-ubuntu-hybrid/gpg'
gpg: Signature made April  6, 2019 at 04:15:08 AM UTC using RSA key ID 476CF100
gpg: Good signature from "Ubuntu Cloud Image Builder (Canonical Internal Cloud Image Builder) <ubuntu-cloudbuilder-noreply@canonical.com>"
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 4A3C E3CD 565D 7EB5 C810  E2B9 7FF3 F408 476C F100
x ubuntu-certified-18.04-20190405-zfs.gz, 352819604 bytes, 689101 tape blocks
x ubuntu-certified-18.04-20190405.json, 1090 bytes, 3 tape blocks
x smartos_image.pkgs, 24343 bytes, 48 tape blocks
Installing ubuntu-certified-18.04-20190405 as d0d07ca4-1445-4d6c-bcb1-6434b9069dbe
Installing image d0d07ca4-1445-4d6c-bcb1-6434b9069dbe (ubuntu-certified-18.04@20190405)
...1445-4d6c-bcb1-6434b9069dbe [=======================================================>] 100% 336.47MB   4.69MB/s  1m11s
Installed image d0d07ca4-1445-4d6c-bcb1-6434b9069dbe (ubuntu-certified-18.04@20190405)
Creating 98447fd9-9ece-ce90-ea2d-fc4e83b1ad16 using image d0d07ca4-1445-4d6c-bcb1-6434b9069dbe
Successfully created VM 98447fd9-9ece-ce90-ea2d-fc4e83b1ad16
Inheriting from origin image d0d07ca4-1445-4d6c-bcb1-6434b9069dbe (ubuntu-certified-18.04 20190405)
Manifest:
    {
      "v": 2,
      "uuid": "69442506-e8ec-4ba4-b6a5-8f857db716b2",
      "name": "ubuntu-certified-18.04",
      "version": "20190405.0.1",
      "owner": "00000000-0000-0000-0000-000000000000",
      "public": false,
      "type": "zvol",
      "cpu_type": "host",
      "description": "Ubuntu 18.04.2 LTS (20190405 64-bit). Certified Ubuntu Server Cloud Image from Canonical.",
      "os": "linux",
      "homepage": "https://docs.joyent.com/images/linux/ubuntu-certified",
      "image_size": 10240,
      "tags": {
        "default_user": "ubuntu",
        "role": "os"
      },
      "files": [],
      "requirements": {
        "min_platform": {
          "7.0": "20150929T232348Z"
        },
        "networks": [
          {
            "name": "net0",
            "description": "public"
          }
        ],
        "ssh_key": true,
        "brand": "kvm"
      },
      "disk_driver": "virtio",
      "nic_driver": "virtio",
      "urn": "sdc:canonical:ubuntu-certified-18.04:20190405"
    }
Snapshotting VM "98447fd9-9ece-ce90-ea2d-fc4e83b1ad16" to @imgadm-create-pre-prepare
Preparing VM 98447fd9-9ece-ce90-ea2d-fc4e83b1ad16 (starting it)
Prepare script is running
Prepare script succeeded
Prepare script stopped VM 98447fd9-9ece-ce90-ea2d-fc4e83b1ad16
Snapshotting to "zones/98447fd9-9ece-ce90-ea2d-fc4e83b1ad16-disk0@final"
Sending image file to "/zones/root/ubuntu-hybrid/mi-ubuntu-hybrid/out/ubuntu-certified-18.04-20190405.0.1.zvol"
Saving manifest to "/zones/root/ubuntu-hybrid/mi-ubuntu-hybrid/out/ubuntu-certified-18.04-20190405.0.1.imgmanifest"
Rollback VM 98447fd9-9ece-ce90-ea2d-fc4e83b1ad16 to pre-prepare snapshot (cleanup)
Successfully deleted VM 98447fd9-9ece-ce90-ea2d-fc4e83b1ad16
Deleted image d0d07ca4-1445-4d6c-bcb1-6434b9069dbe

Successfully created image ubuntu-certified-18.04@20190405.0.1
     zvol: /zones/root/ubuntu-hybrid/mi-ubuntu-hybrid/out/ubuntu-certified-18.04-20190405.0.1.zvol.gz
 manifest: /zones/root/ubuntu-hybrid/mi-ubuntu-hybrid/out/ubuntu-certified-18.04-20190405.0.1.imgmanifest
```

## Testing

To verify that the image works as intended, manual testing is required until
such a time as automated testing is developed.

### Review `out/*.imgmanifest`

Verify that:

  - `image_size` is an integer
  - `requirements.brand` does not exist

### Install the image

```
# imgadm install -m out/*.imgmanifest -f out/*.zvol.gz
...
Installed image 70d88af3-1feb-47a2-9c69-5041e0355cd1 (ubuntu-certified-18.04@20190514.1)
```

### Install an instance and verify basic console functionality

This is intended to verify that the console works, not that the VM is in a
healthy state.  Verifying overall health is beyond the scope of this testing.

Install an instance using the image created and installed above.  This same
process will be repeated for bhyve and kvm brands.

```
# uuid=70d88af3-1feb-47a2-9c69-5041e0355cd1
# sshkey="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABA..."
# brand=bhyve

# vmadm create <<EOF
{
  "alias": "$brand-test",
  "hostname": "$brand-test",
  "brand": "$brand",
  "resolvers": [
    "1.1.1.1"
  ],
  "ram": "1024",
  "vcpus": "2",
  "nics": [
    {
      "nic_tag": "admin",
      "ip": "10.88.88.221",
      "netmask": "255.255.255.0",
      "gateway": "10.88.88.2",
      "model": "virtio",
      "primary": true
    }
  ],
  "disks": [
    {
      "image_uuid": "$uuid",
      "boot": true,
      "model": "virtio"
    }
  ],
  "customer_metadata": {
    "root_authorized_keys": "$sshkey"
  }
}
EOF
Successfully created VM 558dd8b6-4141-c20f-a3b8-8d50ec9754e2
```

Now that the instance is running, log in via ssh.

```
$ ssh ubuntu@10.88.88.221
The authenticity of host '10.88.88.221 (10.88.88.221)' can't be established.
ECDSA key fingerprint is SHA256:j/Zmgl9/Mu+mPyj0yZh/ayWPvtvdC3DDQkYlKifWHX0.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.88.88.221' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 18.04.2 LTS (GNU/Linux 4.15.0-50-generic x86_64)
Certified Ubuntu Cloud Image
...
ubuntu@bhyve-test:~$
```

In another window, connect to the console

```
# vmadm console 558dd8b6-4141-c20f-a3b8-8d50ec9754e2
[Connected to zone '558dd8b6-4141-c20f-a3b8-8d50ec9754e2' console]

Ubuntu 18.04.2 LTS bhyve-test ttyS0

bhyve-test login:
```

From the ssh session, reboot.  After the screen clears, hit `ESC`.  The grub
menu should appear in the console window

```
ubuntu@bhyve-test:~$ sudo reboot
Connection to 10.88.88.221 closed by remote host.
Connection to 10.88.88.221 closed.
```

After hitting `ESC` in the console window:

```
                             GNU GRUB  version 2.02

 +----------------------------------------------------------------------------+
 |*Ubuntu                                                                     |
 | Advanced options for Ubuntu                                                |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 |                                                                            |
 +----------------------------------------------------------------------------+

      Use the ^ and v keys to select which entry is highlighted.
      Press enter to boot the selected OS, `e' to edit the commands
      before booting or `c' for a command-line.
```

Verify that the console takes input by using the arrow keys to cycle though menu
items.  While Ubuntu is selected, press `Enter` to boot the VM.  On the console
you should boot messages followed by the console login prompt.

```
[    0.000000] Linux version 4.15.0-50-generic (buildd@lcy01-amd64-013) (gcc version 7.3.0 (Ubuntu 7.3.0-16ubuntu3)) #54-Ubuntu SMP Mon May 6 18:46:08 UTC 2019 (Ubuntu 4.15.0-50.54-generic 4.15.18)
[    0.000000] Command line: BOOT_IMAGE=/boot/vmlinuz-4.15.0-50-generic root=UUID=941bcda5-7c79-4bc8-88bd-b6dd6961fc39 ro console=ttyS0,115200n8 console=tty0 tsc=reliable earlyprintk
...
Ubuntu 18.04.2 LTS bhyve-test ttyS0

bhyve-test login:
```

Detach from the console (`^].` for bhyve, `^]` for kvm) and delete this
instance.

### Repeat with the other brand

Repeat the previous section with the other brand (switch `bhyve` to `kvm`).

### Check the image with an unrelated VM

In this test, we will use a CentOS image to install a bhyve instance. The second
disk in this instance will be a clone of the image.  This will cause the Ubuntu
image to be mounted at `/data` for easy inspection and no chance that the first
boot process will overwrite cruft that was left in the new image.

```
# centosuuid=1f11188a-c71c-11e8-83d4-370e2a698b16
# uuid=70d88af3-1feb-47a2-9c69-5041e0355cd1
# sshkey="ssh-rsa AAAAB3NzaC1yc2EAAAADAQABA..."
# vmadm create <<EOF
{
  "alias": "test-image",
  "hostname": "test-image",
  "brand": "bhyve",
  "resolvers": [
    "8.8.8.8",
    "8.8.4.4"
  ],
  "ram": "1024",
  "vcpus": "2",
  "nics": [
    {
      "nic_tag": "admin",
      "ips": ["10.88.88.216/24"],
      "gateways": ["10.88.88.2"],
      "model": "virtio",
      "primary": true
    }
  ],
  "disks": [
    {
      "image_uuid": "$centosuuid",
      "boot": true,
      "model": "virtio"
    },
    {
      "image_uuid": "$uuid",
      "model": "virtio"
    }
  ],
  "customer_metadata": {
    "root_authorized_keys": "$sshkey"
  }
}
EOF
Successfully created VM f879ad9a-5758-46a8-dc27-d107483d6684
```

Login to the instance via ssh.

```
$ ssh root@10.88.88.216
The authenticity of host '10.88.88.216 (10.88.88.216)' can't be established.
ECDSA key fingerprint is SHA256:EcMLp7UCqk8+yPwHqcR98B7giHX0gbx7ilUQVY4KqjI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.88.88.216' (ECDSA) to the list of known hosts.
   __        .                   .
 _|  |_      | .-. .  . .-. :--. |-
|_    _|     ;|   ||  |(.-' |  | |
  |__|   `--'  `-' `;-| `-' '  ' `-'
                   /  ;  Instance (CentOS 7.5 20181003)
                   `-'   https://docs.joyent.com/images/linux/centos

[root@test-image ~]# df -h /data
Filesystem      Size  Used Avail Use% Mounted on
/dev/vdb1       7.3G  1.1G  6.2G  15% /data
[root@test-image ~]#
```

Examine each file in /data that has been modified in the past day, looking for
left over ssh keys, IP addresses, etc.  One way to do that is:

```
# cd /data
# find `ls -A` -type f -mtime -1 -size +0 | sort | while read file ; do echo cat $file ;done| bash -x 2>&1 | less
```

The less command `/^\+.*` (slash caret backslash plus dot asterisk) will make
it fairly easy to find each each file in the output.
