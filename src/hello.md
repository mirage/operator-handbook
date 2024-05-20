# Hello, networked world

In this chapter we will install everything necessary to run a unikernel, run the
unikernel, and communicate with it from the host system.

## Tender

To get started with a networking unikernel, you first need to get the solo5
tender. This is the executable that runs in the host operating system, allocates
the resources for the unikernel, and handles the hypercalls. Commonly used is
qemu, or bhyve - but these are pretty large binaries including emulation for
e.g. a floppy disk drive. We have very minimal binaries to run as tender.

### Debian and Ubuntu

For Debian and Ubuntu systems, we provide package repositories. Browse the
[apt package repository](https://apt.robur.coop/dists/) whether your
distribution is periodically built, and add it to `/etc/apt/sources.list`. We
use "ubuntu-20.04" as an example. Please
[open an issue](https://github.com/mirage/operator-handbook/issues) if your
distribution is not supported.

```
$ curl -fsSL https://apt.robur.coop/gpg.pub | gpg --dearmor > /usr/share/keyrings/apt.robur.coop.gpg
$ echo "deb [signed-by=/usr/share/keyrings/apt.robur.coop.gpg] https://apt.robur.coop ubuntu-20.04 main" > /etc/apt/sources.list.d/robur.list # replace ubuntu-20.04 with e.g. debian-11 on a debian buster machine
$ apt update
$ apt install solo5
```

### FreeBSD

We also provide a package repository for FreeBSD. Please
[open an issue](https://github.com/mirage/operator-handbook/issues) if your
release is missing.

```
$ fetch -o /usr/local/etc/pkg/robur.pub https://pkg.robur.coop/repo.pub # download RSA public key
$ echo 'robur: {
  url: "https://pkg.robur.coop/${ABI}",
  mirror_type: "srv",
  signature_type: "pubkey",
  pubkey: "/usr/local/etc/pkg/robur.pub",
  enabled: yes
}' > /usr/local/etc/pkg/repos/robur.conf # Check https://pkg.robur.coop which ABI are available
$ pkg update
$ pkg install solo5
```

### Other distributions

For other distributions and systems we do not (yet?) provide binary packages.
You may be able to find a "solo5" packate as part of your distribution.

### Source installation

You can also download the
[latest source release](https://github.com/Solo5/solo5/releases)
and compile it (the latest release is 0.8.1 at the time of writing):

```
$ curl -o solo5-v0.8.1.tar.gz -fsSL https://github.com/Solo5/solo5/releases/download/v0.8.1/solo5-v0.8.1.tar.gz
$ tar xzf solo5-v0.8.1.tar.gz
$ cd solo5-v0.8.1
$ ./configure.sh
$ make V=1
$ sudo make install
```

## Get your Unikernel

Download the [basic website](https://builds.robur.coop/job/static-website/build/latest)
unikernel binary.

```
$ curl -o website.hvt -fsSL https://builds.robur.coop/job/static-website/build/latest/main-binary
```

## Run your unikernel

The unikernel requires a network interface for communication. You can think of
it as a network cable that you need to plug into the unikernel. As usual, a
cable has two ends -- one that we plug into the unikernel, and the other we
have at the host system. On Linux/Unix systems, such virtual cables are
implemented by a network device called `tap`. You need to create one on the
host system, and configure it, to communicate with the unikernel guest system,

Linux:
```
$ ip tuntap add tap0 mode tap
$ ip tuntap set dev tap0 up
```

FreeBSD:
```
$ sysctl net.link.tap.up_on_open=1
$ ifconfig tap create #will return the device name, i.e. tap0
```

Executing the unikernel:

```
$ sudo solo5-hvt --net:service=tap0 -- website.hvt
```

You should see the output of the unikernel starting. To communicate to the
website unikernel, you can use a web browser. But hold on, we first need to
configure IP connectivity on the host system. We do that by assigning an IP
address to the configured tap interface:

Linux:
```
$ ip addr add 10.0.0.1/24 dev tap0
```

FreeBSD:
```
$ ifconfig tap0 10.0.0.1/24
```

From the command-line, you should now be able to communicate with the unikernel:

```
$ ping -c 2 10.0.0.2
64 bytes from 10.0.0.2: icmp_seq=0 ttl=64 time=0.052 ms
64 bytes from 10.0.0.2: icmp_seq=1 ttl=64 time=0.165 ms
```

And from the web browser as well, type "http://10.0.0.2" as URL.

## Communication to/from external Internet

What are post-routing and pre-routing rules and why do I need them?

note: pre-routing only on external traffic (sysctl & post-routing rule) vs local


How can the unikernel communicate to the external world? An example may be the traceroute unikernel

How can the external world communicate to the unikernel?
