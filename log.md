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
