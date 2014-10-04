# nsenter in a can (ubuntu:14.04 fork)

This is a small Docker recipe to build `nsenter` easily and install it in your
system.


## What is `nsenter`?

It is a small tool allowing to `enter` into `n`ame`s`paces. Technically,
it can enter existing namespaces, or spawn a process into a new set of
namespaces. "What are those namespaces you're blabbering about?"
We are talking about [container namespaces].

`nsenter` can do many useful things, but the main reason why I'm so
excited about it is because it lets you [enter into a Docker container].


## Why build `nsenter` in a container?

This is because my preferred distros (Debian and Ubuntu) ship with an
outdated version of `util-linux` (the package that should contain `nsenter`).
Therefore, if you need `nsenter` on those distros, you have to juggle with
APT repository, or compile from source, or… Ain't nobody got time for that.

I'm going to make a very bold assumption: if you landed here, it's because
you want to enter a Docker container. Therefore, you won't mind if my
method to build `nsenter` uses Docker itself.


## How do I install `nsenter` with this?

If you want to install `nsenter` into `/usr/local/bin`, just do this:

    docker run --rm -v /usr/local/bin:/target dockerimages/nsenter

The `dockerimages/nsenter` container will detect that `/target` is a
mountpoint, and it will copy the `nsenter` binary into it.

If you don't trust me, and prefer to extract the `nsenter` binary,
rather than allowing my container to potentially wreak havoc into
your system's `$PATH`, you can also do this:

    docker run --rm dockerimages/nsenter cat /nsenter > /tmp/nsenter

Then do whatever you want with the binary in `/tmp/nsenter`.


##  How do I *use* `nsenter`?

First, figure out the PID of the container you want to enter:

    PID=$(docker inspect --format {{.State.Pid}} <container_name_or_ID>)

Then enter the container:

    nsenter --target $PID --mount --uts --ipc --net --pid


## What's that docker-enter thing?

It's just a small shell script that wraps up the steps described above into
a tiny helper. It takes the name or ID of a container and optionally the name
of a program to execute inside the namespace. If no command is specified a
shell will be invoked instead.

    # list the root filesystem
    docker-enter my_awesome_container ls -la

## docker-enter with boot2docker

If you are using boot2docker, you can use the function below, to:

- install `nsenter` and `docker-enter` into boot2docker's /var/lib/boot2docker/ directory,
  so they survive restarts.
- execute `docker-enter` inside of boot2docker combined with ssh

```
docker-enter() {
  boot2docker ssh '[ -f /var/lib/boot2docker/nsenter ] || docker run --rm -v /var/lib/boot2docker/:/target jpetazzo/nsenter'
  boot2docker ssh sudo /var/lib/boot2docker/docker-enter "$@"
}
```

You can use it directly from your host (OS X/Windows), no need to ssh into boot2docker.

## Caveats

- This only works on Intel 64 bits platforms. Arguably, this is the
  only officially supported platform for Docker; so it's not a big deal.
  As soon as the Docker registry supports other architectures, I will
  see how to build `nsenter` for those!
- `nsenter` still needs to run from the host; it cannot run inside a
  container (yet).


# Copyright 2014 Frank Lemanschik Direkt SPEED 
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
# http://www.apache.org/licenses/LICENSE-2.0
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[container namespaces]: http://blog.dotcloud.com/under-the-hood-linux-kernels-on-dotcloud-part
[enter into a Docker container]: http://jpetazzo.github.io/2014/03/23/lxc-attach-nsinit-nsenter-docker-0-9/
[Debugging a Docker container]: http://blog.loof.fr/2014/06/debugging-docker-container.html
[Nicolas De Loof]: https://twitter.com/ndeloof
