<!-- vim: set sw=2 sts=2 et ai colorcolumn=73 tw=72 fo+=a: -->

# Wed May 19

We're in the community bonding period, so I'm trying to resist the
temptation to actually sit down and start writing code, but I can at
least use this time to do some research and other preparation, so I can
hit the ground running on June 7.

- Making sure my toolchain works: Compiled virtiofsd-rs and ran a VM
  talking to it.
  - The interesting bit here is that my primary machine runs macOS, but
    obviously I need a Linux host for testing. My current solution is to
    compile and run on an old laptop running Linux, with the source code
    mounted from my main machine over sshfs. This works well enough, but
    the old laptop's a little slow (its battery failed and now its CPU
    won't go above 750 MHz), so if I have time before the coding period
    starts I'd like to either get cross-compilation working, install
    Linux on a faster machine, or set up a nested VM on my primary macOS
    machine.  In any case, I've got something that works well enough, so
    I know I won't get blocked by toolchain issues now.
  - The guest OS isn't too interesting—I really just need a Linux kernel
    and enough of a userspace to poke at it comfortably. Since I'm using
    NixOS as the host, I went with its `nixos-rebuild build-vm` command,
    which gives me a script to launch a fairly lightweight QEMU VM, and
    lets me pass in any additional parameters I want to give QEMU. This
    saves me the trouble of getting Linux installed on a disk image (but
    I'll probably end up doing that at some point to test compatibility
    with other distros).
  - I could read from the virtiofs mount, but not write to it. I'm not
    too concerned about that—I just wanted to make sure I could compile
    and run a Rust vhost-user device, and I've done that—but I'll
    investigate if I have some spare time during the community bonding
    period.
- Some reading up on the SCSI protocol
  - Found this very helpful introduction:
    https://www.devever.net/~hl/scsi

Next: Look into QEMU's tracing feature and see if I can get it to print
out a log of commands, so I can get an intuitive sense of how a SCSI
session goes.

# Thu May 20

Here's a list of every SCSI command number issued by a VM in the process
of booting with our disk attached, mounting an ext4 file system, reading
a file, touching a file, and shutting down, in order of first
invocation:

- command 160: REPORT LUNS
- command 18: INQUIRY
- command 0: TEST UNIT READY
- command 3: REQUEST SENSE
- command 37: READ CAPACITY(10)
- command 90: MODE SENSE(10)
- command 40: READ(10)
- command 158: SERVICE ACTION IN(16)
- command 26: MODE SENSE(6)
- command 163: MAINTENANCE IN
- command 42: WRITE(10)
- command 53: SYNCHRONIZE CACHE(10)

Generated by using these commands, then manually looking up SCSI command
names:

```sh
./result/bin/run-nixos-vm -drive file=$(pwd)/scsi.img,if=none,id=scsi,index=2,werror=report -device virtio-scsi-pci -device scsi-hd,drive=scsi -trace 'scsi_req_parsed' 2>&1 | tee /tmp/trace.out
# awk command: order-preserving uniq, stolen from stack exchange
cat /tmp/trace.out | sed 's/.*tag [0-9]*//' | sed s/dir.*// | awk '!x[$0]++'
```

If we tell QEMU to make the disk read-only (and don't attempt to write),
the guest doesn't try to issue commands 42 or 53, but the other used
commands are identical.

# Tue May 25

Turns out, I'm not anywhere near patient enough to keep myself from
coding until June 7. I've started writing some code, and gotten as far
as receiving and returning a REPORT LUNS command from the BIOS. (Turns
out, QEMU's SeaBIOS speaks virtio-scsi, so I don't even have to boot to
start testing :).) The code at the moment is a huge hack that will blow
up if the slightest thing goes wrong; I really need to sit down and
figure out what abstractions I want to build for myself.

# Wed May 26

Spent the day (I use that word here quite generously--this went
embarassingly far past midnight) chasing some nasty bugs:

- Declaring support for `VhostUserProtocolFeatures::SLAVE_REQ`, but not
  implementing `set_slave_req_fd()`, causes QEMU to print out a
  (harmless, it seems) erorr about an unexpected EOF.
- Linux seems to insist on requesting the VIRTIO_SCSI_F_CHANGE feature
  bit, even if I don't declare support for it - this causes
  vhost-user-backend to error out with `HandleRequest(InvalidParam)`,
  which is about as helpful as it sounds. Not sure what's going on here
  (seems like a Linux bug, but it seems far more likely I'm missing
  something). In any case, declaring support for that feature seems to
  make the problem go away, and I think we can do so for free (it's
  about reporting changes to the configuration of a LUN; I don't think
  I'll be supporting such changes any time soon, so there's nothing to
  do).


