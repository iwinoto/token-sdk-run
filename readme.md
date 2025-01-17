# Learnings of deploying `fabric-samples/token-sdk`
Watch the creator (arne.rutjesisc@nl.ibm.com) of this sample run through a [demo](https://www.youtube.com/live/PX9SDva97vQ?si=szRCcHoUVdmvpKnP&t=1899).

I have only gotten this to work on MacOS. I have tried on Linux (Ubuntu 24.10 and Alpine) and both have failed on installing the `tokengen` utility.

## Installing `tokengen` utility
There was some issues with installation of `tokengen` utility. This utility is only used by the token-sdk sample application set-up. It is not required for a token network.

On MacOS, if there issues with `tokengen` installation, try to clean up the golang cache and module cache then try to reinstall the tool. Use the following commands to clean the golang cache:

```
go clean --cache
go clean --modcache
```

If it still does not work, make sure the golang build dependencies are installed. These are:
```
g++ gcc libc6-dev make pkg-config
```

## Starting Fabric sample network
This only works cleanly when the fabric binaries are in the root directory of `fabric-samples`. If they are stored in any other location the `test-network` scripts will not work, even if the binaries location are in `$PATH`.

I think the correct method is to set `TEST_NETWORK_HOME` environment variable with the location of the fabric binaries that are installed with `install-fabric.sh`. I think this will make sure the binaries look in the right location for config files and other resources.
This is [noted in the `token-sdk/readme.md`](https://github.com/hyperledger/fabric-samples/tree/main/token-sdk#:~:text=Note%3A%20you%20can%20run%20this%20code%20from%20anywhere.%20If%20you%20are%20not%20running%20it%20from%20the%20fabric%2Dsamples/token%2Dsdk%20folder%2C%20also%20set%20the%20following%20environment%20variable%3A).

## Using podman
Making `test-network/network.sh` use podman directly causes channel join to fail in `network.sh up createChannel`.

We have to make this work with docker. We do this by creating a symlink from docker.sock to podman.sock. If running podman rootless, add the following to `~/.profile`:
```
ln -s $XDG_RUNTIME_DIR/podman/podman.sock $XDG_RUNTIME_DIR/docker.sock
export DOCKER_HOST=$XDG_RUNTIME_DIR/docker.sock
```

If running podman as root (not recommended for container security reasons) add the following to a profile script in `/etc/profile.d` (we could call it `/etc/profile.d/podman.sh`).
```
ln -s /run/podman/podman.sock /run/docker.sock
```

In this case, there is no need to set the `DOCKER_HOST` environment variable, since this is the default location for the docker socket.

## Building application nodes (Auditor, Issuer, Owner images)
Building the token network application nodes fails when using go version 1.23.x. The `go.mod` files for each application node source specifies a go version (1.22.0). The container image builds runs `go build` which enforces the installed go version matches the modules declaration. This occurs when bringing up the token network by the `docker-compose` file. An example error output is below:

```
------
 > [auditor builder 5/7] RUN go mod download:
0.049 go: errors parsing go.mod:
[+] Running 0/1pp/go.mod:3: invalid go version '1.22.0': must match format 1.23
 ⠙ Service auditor  Building                                                 2.1s
failed to solve: process "/bin/sh -c go mod download" did not complete successfully: exit code: 1
```

The error output shows the application node that is being built and is producing the error in `[auditor builder 5/7]`. In this case the error is in the `auditor` node. In the end I had to fix the error for all nodes. For each token network node, I edited the `go.mod` to update the go version statement from `1.22.0` to `1.23`.

The line in `go.mod` is supposed to specify the minimum go version to build with. So it should have worked with version 1.23.4 which was installed. So there should be a neater solution to this error.