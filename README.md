# Argo CD in a nutshell

Zero-conf, repeatable
[Argo CD](https://argoproj.github.io/argo-cd/)
environments for demoing, development purposes or troubleshooting (i.e. to
reproduce an issue on a clean environment) using
[Vagrant](https://www.vagrantup.com/)
and
[k3s](https://k3s.io/)
in a single-node setup.

Please be aware that this is work in progress, and mainly serves for my own
purposes. I thought it could be useful enough to share, however, so here it
is.

## Getting started

You will need
[Vagrant](https://www.vagrantup.com/)
and
[VirtualBox](https://www.virtualbox.org/)
installed on your system. Also, this box will set-up a private network with
CIDR range `192.168.254.0/24`, so make sure you don't have this already
setup on your host, or have a route to an already existing net or host in
that range.

Then:

```bash
git clone https://github.com/jannfis/argocd-nutshell
cd argocd-nutshell
vagrant up
```

This will fire up a VM with the default variant of Argo CD up & running. This
can take a couple of minutes, depending on your network speed and general
computer performance specs.

To make sure everything is up & running, `ssh` into the box and check the pods
status:

```shell
$ vagrant ssh
vagrant@argocd-nutshell:~$ kubectl get pods
NAME                                  READY   STATUS    RESTARTS   AGE
argocd-redis-6fb68d9df5-sx7xh         1/1     Running   0          4m30s
argocd-dex-server-748c65b578-kqlpp    1/1     Running   0          4m30s
argocd-application-controller-0       1/1     Running   0          4m30s
argocd-repo-server-64f4ddf469-mbdzn   1/1     Running   0          4m30s
argocd-server-846cf6844-9dvcl         1/1     Running   0          4m30s
```

If all pods are running correctly, you can then access the web UI by visiting

```shell
https://192.168.254.100
```

The default username is `admin`, and the default password is `admin` as well.

## Teardown

When you had enough testing, simply run

```bash
vagrant destroy -f
```

within the directory that contains the `Vagrantfile`.

## What's in it?

On top of the already mentioned K3s cluster and a default installation
of Argo CD with minor customisation (i.e. service of type LoadBalancer and
an admin password set), the following is currently included after the box
has been provisioned:

* `kubectl` pre-configured and with shell completion set-up
* `kubectx` and `kubens` too, also with shell completion set-up
* a `kustomize` binary in `/usr/local/bin` (version 3.9.1)
* The latest released `argocd` CLI, ready to use logged into the Argo CD
  instance

## Installing other Argo CD versions

You can override the Argo CD version installed by the `default` variant by
setting some environment variables before running `vagrant up`:

* `ARGOCD_VERSION`: This lets you specify another version of the manifests
  to install. This can be either a tag name (such as `v1.8.2` or `stable`),
  or use `HEAD` to use the latest manifests from `master` branch.

* `ARGOCD_CLI_VERSION`: Lets you specify the release tag of the Argo CD CLI
  to install. The special value `latest` (which is also the default) will
  look up the latest released version of the CLI from GitHub and install it.

* `ARGOCD_IMAGE`: Lets you override the Argo CD container image to use with
  the manifests. This must be the full path and tag to the image, e.g.
  `quay.io/argoproj/argocd:v1.8.2`. By default, the images as defined in the
  manifests will be used.

## Customization

You can provision custom boxes by doing the following:

* Create a new YAML configuration in the `config` directory. Have a look at
  `config/default.yaml`. Consider all variables mandatory. Be sure to
  change `argocd.variant` variable to the name of your new variant, for
  example `myvariant`.

* The Argo CD manifests are rendered via Kustomize. Have a look at the
  `variants/default` directory for the default variant that is installed.
  It contains Kustomize resources to build the Argo CD manifests. You can
  copy its contents to a new folder with the same name as your new variant.
  Be aware that the Kustomize resources will be templated by Ansible, and
  cannot be rendered using `kustomize` as-is. To troubleshoot, use
  `vagrant provision` on changes, and find the final resources on the box
  within the `/kustomize` directory.

* Set `ARGOCD_VARIANT` to the name of the new variant before running `vagrant
  up`, i.e.

  ```bash
  ARGOCD_VARIANT=myvariant vagrant up
  ```

And then hope it will work out.

## Status

This has just been born, and is not ready for general consumption. Feel free
to use it in the default configuration. Expect to hack on it. It has lots
and lots of rough edges and pitfalls.

**YMMV.**
