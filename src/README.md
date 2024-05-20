# Introduction

We will introduce MirageOS, the target audience, and the scope of this handbook.

## MirageOS

MirageOS is a library operating system, where for each service a separate
binary - a so-called *unikernel* - is used. MirageOS is not a generic purpose
operating system. Each unikernel has a single purpose and is executed as a
virtual machine.

Each unikernel is tailored to its need, and only contains the pieces of code
that are strictly required for its operation: our firewall unikernel doesn't
include user management, a file system. The firewall is only a single process:
there is no process management and no userland vs kernel space process
switching, and also no virtual memory included.

A MirageOS unikernel is minimal in size - this means that the resource
consumption is low since only a tiny binary is kept in memory, and no
unnecessary code is executed (i.e. a cronjob indexing the files on your file
system).

In addition, MirageOS unikernels are developed in the memory-safe and statically
typed programming language [OCaml](https://ocaml.org), which catches lots of
bugs at compile time (static type system) and reduces the attack vectors
(spatial and temporal memory safety issues - buffer overflow and underflows).

It is crucial to have a programming language that enforces memory safety in the
setting with single address space, otherwise an attacker (from a bug in one
library) could access all the memory of the other libraries. MirageOS comes
with some C code, which is compiled with defense-in-depth features.

The benefits of MirageOS unikernels are (a) lower resource consumption,
(b) smaller attack surface, (c) avoidance of memory safety attacks, and (d)
reduced operational complexity.

## Target audience and requirements

The reader of this handbook are persons who want to operate MirageOS unikernels.
There won't be any OCaml code in this handbook. You don't need an OCaml
compiler for this handbook. Only the operational aspects are discussed, for
example how to setup networking.

Of course, you as an OCaml or MirageOS developer can use your custom unikernel
instead of the stock ones provided.

You will need a hypervisor, supported are [Linux kvm](https://linux-kvm.org/),
[FreeBSD BHyve](https://bhyve.org/), and
[OpenBSD VMM](https://man.openbsd.org/vmm).

It is suggested to have a x86 laptop or server with one of the operating systems
and the required hardware support for virtualization.

Within the handbook, we will link to reproducible binaries of the specific
unikernels, available from [builds.robur.coop](https://builds.robur.coop).
The unikernels are built on a daily basis using the latest releases of OCaml
packages.

Our focus on running MirageOS as a hardware-supported virtual machine (also
named `hvt`) has multiple reasons:
- from a security point of view, it leverages the CPU virtualization features,
  and only uses a tiny API to the host operating system,
- we are able to ship binaries that run on either of the hypervisors, no need
  for custom binaries depending on the hypervisor.
- it is our preferred target, and we run it since nearly a decade!

## Other MirageOS targets

MirageOS supports more targets we will only discuss briefly in this section.
If you want to read more, take a look at the
[Solo5 documentation](https://github.com/Solo5/solo5/blob/master/docs/building.md#supported-targets)

### Unix

The default target of MirageOS for development is unix -- this uses the
normal network stack by default (via the sockets API). It is nice for
development, but security-wise it is not a good solution for deployment.

### [Linux seccomp](https://en.wikipedia.org/wiki/Seccomp)

The Linux seccomp target uses restrictive seccomp rules and allows only a few
system calls to be executed. This does not require a hypervisor, and can be
executed similar to a container. From a security perspective, the attack surface
is slightly bigger - it is the seccomp implementation. From the network
operations perspective, this is the same as what we discuss in this handbook.
Since we don't provide pre-built unikernel binaries for seccomp, you'll need
to compile them yourself. And instead of executing `solo5-hvt`, you need to run
`solo5-spt`.

### [Xen](https://xenproject.org/)

MirageOS was originally developed by some people who also worked on Xen. Xen is
a great hypervisor, but very different from the modern ones, which rely on CPU
features. The Xen target is used with MirageOS, but the network setup is very
different, thus we won't focus on that in this handbook.

### [Muen](https://muen.sk) separation kernel

The Muen separation kernel is developed in Ada/SPARK - which includes a
verification framework. It is a great project, and if you plan to deploy
high-assurance solutions, you should dive deeper into Muen. Since its operation
and configuration is custom, we won't discuss it further in this handbook.

### [Virtio](https://wiki.libvirt.org/Virtio.html)

Virtio allows to run virtual machines on the cloud, where the hypervisor is
managed for you by some company. This allows to deploy MirageOS onto the cloud.
Since the configuration again is very different, we don't go into further
details. But we plan to write a chapter on virtio for this handbook.

## Conclusion

We briefly discussed the different MirageOS backends, and have the focus on the
hypervisor backend.

This handbook is not complete, suggestions are welcome. The initial chapters is
focused on network setups that were encountered when testing unikernels, e.g.
at our annual [MirageOS retreat](https://retreat.mirage.io).

Please file suggestions as an
[issue](https://github.com/mirage/operator-handbook) on GitHub, or a pull
request.
