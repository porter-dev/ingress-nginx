# ingress-nginx
A repo for building Chainguard's fork of [ingress-nginx](https://github.com/chainguard-forks/ingress-nginx).

## Why?
As you would no doubt be aware, `ingress-nginx` is being deprecated in March. This means we're also not see new releases that fix any CVEs that will inevitably pop up after this deprecation. Luckily Chainguard has decided to bring `ingress-nginx` under the [EmeritOSS programme](https://www.chainguard.dev/unchained/introducing-chainguard-emeritoss) - this means they'll maintain a fork of the upstream `ingress-nginx` repos and [patch CVEs as best as they can](https://www.chainguard.dev/unchained/keeping-ingress-nginx-alive). This allows us a window where we can continue running `ingress-nginx` on customer clusters, whilst working on a [ramp towards moving every customer towards the community HAProxy Ingress Controller](https://www.notion.so/porter-run/HAProxy-Ingress-2f075cbf10de80e5ac4ac76a23253d13?source=copy_link). 

## Using these images

We're currently hosting these built images at [porterhub/controller](https://hub.docker.com/r/porterhub/controller). Due to the way the Makefile for `ingress-nginx` functions, we're stuck with using `controller` as the repo name on Docker Hub, since the Makefile typically expects the owner name to be `ingress-nginx`. This isn't something I'm interested in tweaking atm. The [build process](./.github/workflows/build.yml) pulls in [Chainguard's fork of `ingress-nginx`](https://github.com/chainguard-forks/ingress-nginx), and builds off the `helm-chart-4.12.1` tag and builds multi-arch images. More up-to-date tags will be added gradually.

To reconfigure `ingress-nginx` on a customer's cluster to use this image, simply modify the default `controller.image` block so that it looks like this:

```yaml
controller:
  image:
    registry: docker.io
    image: porterhub/controller
    tag: "v1.12.1"
    digest: null
```

## Helm charts

We've elected to also operate our own mirror for the Helm charts itself. To make things simpler, we've pushed these charts as OCI images to [Docker Hub](https://hub.docker.com/r/porterhub/ingress-nginx). To use these images, all you need to do is point your `helm install` / `helm upgrade` commands at `oci://registry-1.docker.io/porterhub/ingress-nginx`:

```bash
helm upgrade --install ingress-nginx oci://registry-1.docker.io/porterhub/ingress-nginx -n ingress-nginx --version "4.12.1" -f ./ingress-nginx-values.yaml
```

## Supported versions

We currently build the following versions:

| Helm Chart Version | Controller Image Tag | Image Repository | Chart Repository |
|--------------------|---------------------|------------------|------------------|
| 4.11.5             | v1.11.5             | porterhub/controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.12.1             | v1.12.1             | porterhub/controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.12.8             | v1.12.8             | porterhub/controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.13.0             | v1.13.0             | porterhub/controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.13.4             | v1.13.4             | porterhub/controller | oci://registry-1.docker.io/porterhub/ingress-nginx |
| 4.14.0             | v1.14.0             | porterhub/controller | oci://registry-1.docker.io/porterhub/ingress-nginx |

## Running a fresh build

The build workflow is set to be triggered manually, using `workflow_dispatch`. To build a fresh image(assuming there's been an update on Chainguard's end):
1. Navigate to `Actions`, and click on `Build and push ingress-nginx images` on the left sidebar, under `Actions`.
2. Click on `Run workflow`.
3. Hit `Run workflow` and then refresh the page, to see the new run.

In addition, the build workflow is configured to run every Tuesday at 3am UTC.