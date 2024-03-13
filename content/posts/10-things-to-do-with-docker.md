---
title: "10 Things to Do With Docker"
date: 2024-03-12T13:24:47+02:00
draft: true
---
## Operate on root files without sudo password

We all know that docker access __is basically root access__, but examples are critical to understanding. 

{{< highlight bash >}}
$ cat /etc/shadow
cat: /etc/shadow: Permission denied

$ docker run --mount type=bind,source="/etc/",target="/etc-host" \
    --rm --name nosudo -it ubuntu bash -c "cat /etc-host/shadow"
root:!:***********************:::
daemon:***********************:::
bin:*:************************:::
    
{{< / highlight >}}

## Operate on docker itself

This is like docker inside docker (dind), but it doesn't require any privileged permissions, but given the example above, you know...

{{< highlight bash >}}
docker run --rm --name nosudo \
    --mount type=bind,source="/var/run",target="/var-host-run" \
    -it ubuntu bash \
    -c "apt update && apt install -y curl && curl --unix-socket /var-host-run/docker.sock http://localhost/v1.44/containers/json"
{{< / highlight >}}

## Block specific system calls using seccomp filters

A `seccomp` filter is the Linux foundation for all the sandboxes running under Linux, it's possible to do it own your own. 

You download a default seccomp filter from somewhere, like [this](https://raw.githubusercontent.com/docker/labs/master/security/seccomp/seccomp-profiles/default.json), 
then modify the list accordingly. 

Say you don't want `connect()` syscalls

{{< highlight json >}}
{
    "name": "connect",
    "action": "SCMP_ACT_ALLOW",
    "args": []
}
{{< / highlight >}}

Modify to 

{{< highlight json >}}
{
    "name": "connect",
    "action": "SCMP_ACT_ERRNO",
    "args": []
}
{{< / highlight >}}

And now if you do this:  (adding `--security-opts seccomp=profile.json`) 


{{< highlight bash >}}
docker run --rm --name nosudo \
    --security-opt seccomp=profile.json --mount \
    type=bind,source="/var/run",target="/var-host-run" -it ubuntu bash \
    -c "apt update && apt install -y curl && curl --unix-socket /var-host-run/docker.sock http://localhost/v1.44/containers/json"
{{< / highlight >}}


