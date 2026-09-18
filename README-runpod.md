# Runpod's fork of the Moby Project

## Context

Runpod uses [Kata Containers](https://github.com/kata-containers/kata-containers), which increases pod isolation by creating a dedicated micro-vm per workload.
Their [guide on privileged containers](https://github.com/kata-containers/kata-containers/blob/main/docs/how-to/privileged.md#enabling-privileged-containers-without-host-devices) covers the CRI interface (which kubelet uses).

Because moby uses the shim interface and crafts the OCI spec, we need this patch to introduce the `privileged-without-host-devices` parameter.

## Existing changes

We currently have two commits (one will be added for this document) that are reachable from `29.8.1-rp` minus anything reachable from `docker-v29.8.1`:

```console
❯ git log docker-v29.8.1..29.8.1-rp --oneline
16656caae0 (HEAD -> 29.8.1-rp, origin/29.8.1-rp, origin) daemon: test privileged-without-host-devices
b2232fcd62 new security-opt: privileged-without-host-devices
```

The whole delta as one patch can be queried using `git diff docker-v29.8.1..29.8.1-rp`.

## Testing

The `daemon` package does not build on macOS, so the tests run in a container pulled from `mirror.gcr.io`:

```console
❯ docker run --rm -v "$PWD":/src:ro -w /src \
    -e GOFLAGS=-mod=vendor -e GOCACHE=/tmp/gocache -e CGO_ENABLED=0 \
    mirror.gcr.io/library/golang:1.26 \
    go test ./daemon/ -run 'TestParseSecurityOpt$|TestWithDevicesPrivilegedWithoutHostDevices' -v
```

## Future cherry-pick

When the future version lands (let's call it `docker-v29.8.2` for the sake of the example):

### Create the new branch

Create a new runpod branch tracking the desired upstream version

```console
❯ git switch -c 29.8.2-rp docker-v29.8.2
```

### Cherry-pick the changes

```console
❯ git cherry-pick -x docker-v29.8.1..29.8.1-rp
```

### Potentially update this document

Update the refs and the commit list above to match the new version.
