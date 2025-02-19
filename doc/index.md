[![LXD](https://linuxcontainers.org/static/img/containers.png)](https://linuxcontainers.org/lxd)
# LXD
LXD is a next generation system container and virtual machine manager.  
It offers a unified user experience around full Linux systems running inside containers or virtual machines.

It's image based with pre-made images available for a [wide number of Linux distributions](https://images.linuxcontainers.org)  
and is built around a very powerful, yet pretty simple, REST API.

To get a better idea of what LXD is and what it does, you can [try it online](https://linuxcontainers.org/lxd/try-it/)!  
Then if you want to run it locally, take a look at our [getting started guide](https://linuxcontainers.org/lxd/getting-started-cli/).

Release announcements can be found here: <https://linuxcontainers.org/lxd/news/>  
And the release tarballs here: <https://linuxcontainers.org/lxd/downloads/>
The documentation is here: <https://linuxcontainers.org/lxd/docs/master/>

## Status
Type                | Service               | Status
---                 | ---                   | ---
CI (client)         | GitHub                | [![Build Status](https://github.com/lxc/lxd/workflows/Client%20build%20and%20unit%20tests/badge.svg)](https://github.com/lxc/lxd/actions)
CI (server)         | Jenkins               | [![Build Status](https://jenkins.linuxcontainers.org/job/lxd-github-commit/badge/icon)](https://jenkins.linuxcontainers.org/job/lxd-github-commit/)
Go documentation    | Godoc                 | [![GoDoc](https://godoc.org/github.com/lxc/lxd/client?status.svg)](https://godoc.org/github.com/lxc/lxd/client)
Static analysis     | GoReport              | [![Go Report Card](https://goreportcard.com/badge/github.com/lxc/lxd)](https://goreportcard.com/report/github.com/lxc/lxd)
Translations        | Weblate               | [![Translation status](https://hosted.weblate.org/widgets/linux-containers/-/svg-badge.svg)](https://hosted.weblate.org/projects/linux-containers/lxd/)
Project status      | CII Best Practices    | [![CII Best Practices](https://bestpractices.coreinfrastructure.org/projects/1086/badge)](https://bestpractices.coreinfrastructure.org/projects/1086)

## Installing LXD from packages
The LXD daemon only works on Linux but the client tool (`lxc`) is available on most platforms.

OS                  | Format                                            | Command
---                 | ---                                               | ---
Linux               | [Snap](https://snapcraft.io/lxd)                  | snap install lxd
Windows             | [Chocolatey](https://chocolatey.org/packages/lxc) | choco install lxc
MacOS               | [Homebrew](https://formulae.brew.sh/formula/lxc)  | brew install lxc

More instructions on installing LXD for a wide variety of Linux distributions and operating systems [can be found on our website](https://linuxcontainers.org/lxd/getting-started-cli/).

## Installing LXD from source
We recommend having the latest versions of liblxc (>= 3.0.0 required)
available for LXD development. Additionally, LXD requires Golang 1.13 or
later to work. On ubuntu, you can get those with:

```bash
sudo apt update
sudo apt install acl attr autoconf dnsmasq-base git golang libacl1-dev libcap-dev liblxc1 liblxc-dev libsqlite3-dev libtool libudev-dev liblz4-dev libuv1-dev make pkg-config rsync squashfs-tools tar tcl xz-utils ebtables
```

Note that when building LXC yourself, ensure to build it with the appropriate
security related libraries installed which our testsuite tests. Again, on
ubuntu, you can get those with:

```bash
sudo apt install libapparmor-dev libseccomp-dev libcap-dev
```

There are a few storage backends for LXD besides the default "directory" backend.
Installing these tools adds a bit to initramfs and may slow down your
host boot, but are needed if you'd like to use a particular backend:

```bash
sudo apt install lvm2 thin-provisioning-tools
sudo apt install btrfs-progs
```

To run the testsuite, you'll also need:

```bash
sudo apt install curl gettext jq sqlite3 uuid-runtime socat
```

### From Source: Building the latest version

These instructions for building from source are suitable for individual developers who want to build the latest version
of LXD, or build a specific release of LXD which may not be offered by their Linux distribution. Source builds for
integration into Linux distributions are not covered here and may be covered in detail in a separate document in the
future.

```bash
git clone https://github.com/lxc/lxd
cd lxd
```

This will download the current development tree of LXD and place you in the source tree.
Then proceed to the instructions below to actually build and install LXD.

### From Source: Building a Release

The LXD release tarballs bundle a complete dependency tree as well as a
local copy of libraft and libdqlite for LXD's database setup.

```bash
tar zxvf lxd-4.18.tar.gz
cd lxd-4.18
```

This will unpack the release tarball and place you inside of the source tree.
Then proceed to the instructions below to actually build and install LXD.

### Starting the Build

The actual building is done by two separate invocations of the Makefile: `make deps` -- which builds libraries required 
by LXD -- and `make`, which builds LXD itself. At the end of `make deps`, a message will be displayed which will specify environment variables that should be set prior to invoking `make`. As new versions of LXD are released, these environment
variable settings may change, so be sure to use the ones displayed at the end of the `make deps` process, as the ones
below (shown for example purposes) may not exactly match what your version of LXD requires:

We recommend having at least 2GB of RAM to allow the build to complete.

```bash
make deps
# Follow the instructions from `make deps` to export the required environment variables.
# For example:
#  export CGO_CFLAGS="${CGO_CFLAGS} -I$(go env GOPATH)/deps/dqlite/include/ -I$(go env GOPATH)/deps/raft/include/"
#  export CGO_LDFLAGS="${CGO_LDFLAGS} -L$(go env GOPATH)/deps/dqlite/.libs/ -L$(go env GOPATH)/deps/raft/.libs/"
#  export LD_LIBRARY_PATH="$(go env GOPATH)/deps/dqlite/.libs/:$(go env GOPATH)/deps/raft/.libs/:${LD_LIBRARY_PATH}"
#  export CGO_LDFLAGS_ALLOW="(-Wl,-wrap,pthread_create)|(-Wl,-z,now)"
make
```

### From Source: Installing

Once the build completes, you simply keep the source tree, add the directory referenced by `$(go env GOPATH)/bin` to
your shell path, and set the `LD_LIBRARY_PATH` variable printed by `make deps` to your environment. This might look
something like this for a `~/.bashrc` file:

```bash
export PATH="${PATH}:$(go env GOPATH)/bin"
export LD_LIBRARY_PATH="$(go env GOPATH)/deps/dqlite/.libs/:$(go env GOPATH)/deps/raft/.libs/:${LD_LIBRARY_PATH}"
```

Now, the `lxd` and `lxc` binaries will be available to you and can be used to set up LXD. The binaries will automatically find and use the dependencies built in `$(go env GOPATH)/deps` thanks to the `LD_LIBRARY_PATH` environment variable.

### Machine Setup
You'll need sub{u,g}ids for root, so that LXD can create the unprivileged containers:

```bash
echo "root:1000000:65536" | sudo tee -a /etc/subuid /etc/subgid
```

Now you can run the daemon (the `--group sudo` bit allows everyone in the `sudo`
group to talk to LXD; you can create your own group if you want):

```bash
sudo -E PATH=${PATH} LD_LIBRARY_PATH=${LD_LIBRARY_PATH} $(go env GOPATH)/bin/lxd --group sudo
```

## Security
LXD, similar to other container and VM managers provides a UNIX socket for local communication.

**WARNING**: Anyone with access to that socket can fully control LXD, which includes
the ability to attach host devices and filesystems, this should
therefore only be given to users who would be trusted with root access
to the host.

When listening on the network, the same API is available on a TLS socket
(HTTPS), specific access on the remote API can be restricted through
Canonical RBAC.


More details are [available here](security.md).

## Getting started with LXD
Now that you have LXD running on your system you can read the [getting started guide](https://linuxcontainers.org/lxd/getting-started-cli/) or go through more examples and configurations in [our documentation](https://linuxcontainers.org/lxd/docs/master/).

## Bug reports
Bug reports and Feature requests can be filed at: <https://github.com/lxc/lxd/issues/new>

## Contributing
Fixes and new features are greatly appreciated but please read our [contributing guidelines](contributing.md) first.

## Support and discussions
### Forum
A discussion forum is available at: <https://discuss.linuxcontainers.org>

### Mailing-lists
We use the LXC mailing-lists for developer and user discussions, you can
find and subscribe to those at: <https://lists.linuxcontainers.org>

### IRC
If you prefer live discussions, you can find us in [#lxc](https://kiwiirc.com/client/irc.libera.chat/#lxc) on irc.libera.chat.

## FAQ
#### How to enable LXD server for remote access?
By default LXD server is not accessible from the networks as it only listens
on a local unix socket. You can make LXD available from the network by specifying
additional addresses to listen to. This is done with the `core.https_address`
config variable.

To see the current server configuration, run:

```bash
lxc config show
```

To set the address to listen to, find out what addresses are available and use
the `config set` command on the server:

```bash
ip addr
lxc config set core.https_address 192.168.1.15
```

#### When I do a `lxc remote add` over https, it asks for a password?
By default, LXD has no password for security reasons, so you can't do a remote
add this way. In order to set a password, do:

```bash
lxc config set core.trust_password SECRET
```

on the host LXD is running on. This will set the remote password that you can
then use to do `lxc remote add`.

You can also access the server without setting a password by copying the client
certificate from `.config/lxc/client.crt` to the server and adding it with:

```bash
lxc config trust add client.crt
```

#### How do I configure LXD storage?
LXD supports btrfs, ceph, directory, lvm and zfs based storage.

First make sure you have the relevant tools for your filesystem of
choice installed on the machine (btrfs-progs, lvm2 or zfsutils-linux).

By default, LXD comes with no configured network or storage.
You can get a basic configuration done with:

```bash
    lxd init
```

`lxd init` supports both directory based storage and ZFS.
If you want something else, you'll need to use the `lxc storage` command:

```bash
lxc storage create default BACKEND [OPTIONS...]
lxc profile device add default root disk path=/ pool=default
```

BACKEND is one of `btrfs`, `ceph`, `dir`, `lvm` or `zfs`.

Unless specified otherwise, LXD will setup loop based storage with a sane default size.

For production environments, you should be using block backed storage
instead both for performance and reliability reasons.

#### How can I live migrate a container using LXD?
Live migration requires a tool installed on both hosts called
[CRIU](https://criu.org), which is available in Ubuntu via:

```bash
sudo apt install criu
```

Then, launch your container with the following,

```bash
lxc launch ubuntu SOME-NAME
sleep 5s # let the container get to an interesting state
lxc move host1:SOME-NAME host2:SOME-NAME
```

And with luck you'll have migrated the container :). Migration is still in
experimental stages and may not work for all workloads. Please report bugs on
lxc-devel, and we can escalate to CRIU lists as necessary.

#### Can I bind mount my home directory in a container?
Yes. This can be done using a disk device:

```bash
lxc config device add container-name home disk source=/home/${USER} path=/home/ubuntu
```

For unprivileged containers, you will also need one of:

 - Pass `shift=true` to the `lxc config device add` call. This depends on `shiftfs` being supported (see `lxc info`)
 - raw.idmap entry (see [Idmaps for user namespace](userns-idmap.md))
 - Recursive POSIX ACLs placed on your home directory

Either of those can be used to allow the user in the container to have working read/write permissions.
When not setting one of those, everything will show up as the overflow uid/gid (65536:65536)
and access to anything that's not world readable will fail.


Privileged containers do not have this issue as all uid/gid inthe container are the same outside.
But that's also the cause of most of the security issues with such privileged containers.

#### How can I run docker inside a LXD container?
In order to run Docker inside a LXD container the `security.nesting` property of the container should be set to `true`. 

```bash
lxc config set <container> security.nesting true
```

Note that LXD containers cannot load kernel modules, so depending on your
Docker configuration you may need to have the needed extra kernel modules
loaded by the host.

You can do so by setting a comma separate list of kernel modules that your container needs with:

```bash
lxc config set <container> linux.kernel_modules <modules>
```

We have also received some reports that creating a `/.dockerenv` file in your
container can help Docker ignore some errors it's getting due to running in a
nested environment.

## Hacking on LXD
### Directly using the REST API
The LXD REST API can be used locally via unauthenticated Unix socket or remotely via SSL encapsulated TCP.

#### Via Unix socket

```bash
curl --unix-socket /var/lib/lxd/unix.socket \
    -H "Content-Type: application/json" \
    -X POST \
    -d @hello-ubuntu.json \
    lxd/1.0/containers
```
or for snap users:

```bash
curl --unix-socket /var/snap/lxd/common/lxd/unix.socket \
    -H "Content-Type: application/json" \
    -X POST \
    -d @hello-ubuntu.json \
    lxd/1.0/containers
```


#### Via TCP
TCP requires some additional configuration and is not enabled by default.

```bash
lxc config set core.https_address "[::]:8443"
```

```bash
curl -k -L \
    --cert ~/.config/lxc/client.crt \
    --key ~/.config/lxc/client.key \
    -H "Content-Type: application/json" \
    -X POST \
    -d @hello-ubuntu.json \
    "https://127.0.0.1:8443/1.0/containers"
```

#### JSON payload
The `hello-ubuntu.json` file referenced above could contain something like:

```json
{
    "name":"some-ubuntu",
    "ephemeral":true,
    "config":{
        "limits.cpu":"2"
    },
    "source": {
        "type":"image",
        "mode":"pull",
        "protocol":"simplestreams",
        "server":"https://cloud-images.ubuntu.com/releases",
        "alias":"18.04"
    }
}
```
