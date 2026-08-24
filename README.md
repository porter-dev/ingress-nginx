[![Manual Builds](https://github.com/porter-dev/ingress-nginx/actions/workflows/build.yml/badge.svg?event=workflow_dispatch)](https://github.com/porter-dev/ingress-nginx/actions/workflows/build.yml)

[![Weekly Builds](https://github.com/porter-dev/ingress-nginx/actions/workflows/build.yml/badge.svg?event=schedule)](https://github.com/porter-dev/ingress-nginx/actions/workflows/build.yml)

# ingress-nginx
A repo for building Chainguard's fork of [ingress-nginx](https://github.com/chainguard-forks/ingress-nginx).

## Why?
As you would no doubt be aware, `ingress-nginx` is being deprecated in March. This means we're also not see new releases that fix any CVEs that will inevitably pop up after this deprecation. Luckily Chainguard has decided to bring `ingress-nginx` under the [EmeritOSS programme](https://www.chainguard.dev/unchained/introducing-chainguard-emeritoss) - this means they'll maintain a fork of the upstream `ingress-nginx` repos and [patch CVEs as best as they can](https://www.chainguard.dev/unchained/keeping-ingress-nginx-alive). This allows us a window where we can continue running `ingress-nginx` on customer clusters, whilst working on a [ramp towards moving every customer towards the community HAProxy Ingress Controller](https://www.notion.so/porter-run/HAProxy-Ingress-2f075cbf10de80e5ac4ac76a23253d13?source=copy_link). 

## Using these images

We're currently hosting these built images at [ghcr.io/porter-dev/ingress-nginx-controller](https://hub.docker.com/r/ghcr.io/porter-dev/ingress-nginx-controller). Due to the way the Makefile for `ingress-nginx` functions, we're stuck with using `controller` as the repo name on Docker Hub, since the Makefile typically expects the owner name to be `ingress-nginx`. This isn't something I'm interested in tweaking atm. The [build process](./.github/workflows/build.yml) pulls in [Chainguard's fork of `ingress-nginx`](https://github.com/chainguard-forks/ingress-nginx), and builds off the `helm-chart-4.12.1` tag and builds multi-arch images. More up-to-date tags will be added gradually.

To reconfigure `ingress-nginx` on a customer's cluster to use this image, simply modify the default `controller.image` block so that it looks like this:

```yaml
controller:
  image:
    registry: docker.io
    image: ghcr.io/porter-dev/ingress-nginx-controller
    tag: "v1.12.1"
    digest: null
```

Alternatively, you can point to GHCR:

```yaml
controller:
  image:
    registry: ghcr.io
    image: porter-dev/ingress-nginx-controller
    tag: "v1.12.1"
    digest: null
```


## Helm charts

We've elected to also operate our own mirror for the Helm charts itself. To make things simpler, we've pushed these charts as OCI images to [Docker Hub](https://hub.docker.com/r/porterhub/ingress-nginx). To use these images, all you need to do is point your `helm install` / `helm upgrade` commands at `oci://registry-1.docker.io/porterhub/ingress-nginx`:

```bash
helm upgrade --install ingress-nginx oci://registry-1.docker.io/porterhub/ingress-nginx -n ingress-nginx --version "4.12.1" -f ./ingress-nginx-values.yaml
```

Our build workflow is responsible for pushing updated OCI packages to Docker Hub every time new images are built.

## Supported versions

We don't pin individual versions anymore. Instead the workflow tracks a set of
**controller minor lines** (the `SUPPORTED_MINORS` knob in [`build.yml`](./.github/workflows/build.yml)),
and on every run the `prepare` job discovers the **latest upstream `controller-v<minor>.<patch>` tag**
for each line and builds it. Because the newest patch always carries the newest
nginx base, this means new controller patches **and** their CVE-patched base
images are picked up automatically — nobody has to edit the matrix when
Chainguard ships a rebuild.

Currently tracked lines: **1.12, 1.14, 1.15**. As of the latest run that
resolves to:

| Helm Chart Version | Controller Image Tag | Base Image Tag | Image Repository | Chart Repository |
|--------------------|---------------------|----------------|------------------|------------------|
| 4.12.8             | v1.12.8             | v1.3.4  | ghcr.io/porter-dev/ingress-nginx-controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.14.5             | v1.14.5             | v2.2.9  | ghcr.io/porter-dev/ingress-nginx-controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.15.10            | v1.15.10            | v2.2.13 | ghcr.io/porter-dev/ingress-nginx-controller | oci://registry-1.docker.io/porterhub/ingress-nginx |

This table is a **snapshot** — the actual versions float to whatever the newest
patch of each line is at build time. The controller image tag and chart version
are derived from the upstream **tag name** (e.g. `controller-v1.15.10` → image
`v1.15.10`, chart `4.15.10`), not the fork's internal `TAG`/`Chart.yaml`, since
Chainguard sometimes leaves those stale. Images are built for `linux/amd64` and
`linux/arm64`. The base image tag is the nginx base (`images/nginx/TAG` in the
fork) that each controller is built on top of; we rebuild it from source and
push it to `porterhub/nginx` (and `ghcr.io/porter-dev/ingress-nginx-nginx`) on
every run, then pin each controller to it **by digest**, so OS/CVE fixes land
even when the controller code itself is unchanged.

To start supporting a new minor line (e.g. `1.16`), add it to `SUPPORTED_MINORS`
in the workflow — that's the only edit ever required.

## Running a fresh build

The build workflow is set to be triggered manually, using `workflow_dispatch`. To build a fresh image(assuming there's been an update on Chainguard's end):
1. Navigate to `Actions`, and click on `Build and push ingress-nginx images` on the left sidebar, under `Actions`.
2. Click on `Run workflow`.
3. Hit `Run workflow` and then refresh the page, to see the new run.

In addition, the build workflow is configured to run every Tuesday at 3am UTC.