# Learnings of deploying `fabric-samples/token-sdk`
Watch the creator of this sample run through a [demo](https://www.youtube.com/live/PX9SDva97vQ?si=szRCcHoUVdmvpKnP&t=1899).

I have only gotten this to work on MacOS. I have tried on Linux (Ubuntu 24.10 and Alpine) and both have failed on installing the `tokengen` utility.

## Installing `tokengen` utility
There was some issues with installation of `tokengen` utility. This utility is only used by the token-sdk sample application set-up. It is not required for a token network.

On MacOS, if there issues with `tokengen` installation, try to clean up the golang cache and module cache then try to reinstall the tool. Use the following commands to clean the golang cache:

```
go clean --cache
go clean --modcache
```

## Starting Fabric sample network
This only works cleanly when the fabric binaries are in the root directory of `fabric-samples`. If they are stored in any other location the `test-network` scripts will not work, even if the binaries location are in `$PATH`.

## Building application nodes (Auditor, Issuer, Owner images)
Building the token network application nodes fails when using go version 1.23.x. The `go.mod` files for each application node source specifies a go version (1.22.0). The container image builds runs `go build` which enforces the installed go version matches the modules declaration. This occurs when bringing up the token network by the `docker-compose` file. An example error output is below:

```
------
 > [auditor builder 5/7] RUN go mod download:
0.049 go: errors parsing go.mod:
[+] Running 0/1pp/go.mod:3: invalid go version '1.23.0': must match format 1.23
 ⠙ Service auditor  Building                                                 2.1s
failed to solve: process "/bin/sh -c go mod download" did not complete successfully: exit code: 1
```

The error output shows the application node that is being built and is producing the error in `[auditor builder 5/7]`. In this case the error is in the `auditor` node. In the end I had to fix the error for all nodes. For each token network node, I edited the `go.mod` to update the go version statement from `1.22.0` to `1.23`.

