# Argo CD example application using Helm Sops

## Requirements

To deploy this example application, you need a running Kubernetes cluster. You can get one using [Minikube](https://minikube.sigs.k8s.io/docs/start/).

You also need to have GnuPG, [Sops](https://github.com/mozilla/sops), [Helm](https://helm.sh/docs/intro/install/) (version 3 and optionally version 2) and [Helm Sops](https://github.com/camptocamp/helm-sops) installed on your workstation.

Helm Sops must be installed to transparently wrap Helm. The recommended setup is to have Helm 3 installed as the `_helm3` binary with a `_helm` symlink pointing to it and Helm 2 installed as the `_helm2` binary. This way, you can easily switch from Helm 3 to Helm 2 as your default Helm version. Helm Sops must then be installed as the `helm` binary with `helm3` and `helm2` symlinks pointing to it. The result should look like this:

```
lrwxrwxrwx.  1 user   user         6 Apr  9 17:25 _helm -> _helm3
-rwxr-xr-x.  1 user   user  40460288 Apr 14 21:42 _helm2
-rwxr-xr-x.  1 user   user  38461440 Mar 12 19:38 _helm3
-rwxr-xr-x.  1 user   user  22390865 Apr  3 18:02 helm
lrwxrwxrwx.  1 user   user         4 Apr  3 18:02 helm2 -> helm
lrwxrwxrwx.  1 user   user         4 Apr  3 18:03 helm3 -> helm
```

Finally, clone this repository and open a terminal at its root.

## Using the example GPG keyring

There is an example GPG keyring in the `gpg-keyring` directory which contains the secret key of an administrator named John Doe (which you will impersonate) and the public key of the Argo CD instance which you're about to install. The Sops configuration file (`.sops.yaml`) in the repository references these GPG keys. Argo CD's private key is configured in the `argocd/secrets.yaml` file.

To make use of this GPG keyring, execute the following command in the terminal.

```sh
export GNUPGHOME="$(realpath gpg-keyring)"
```

Of course, in a real setup, you would generate a dedicated private key for Argo CD and use your own GPG key (or generate a new one) instead of John Doe's.

## Installing Argo CD

Run the following commands in the terminal (ensure Helm 3 is your default Helm version):

```sh
cd argocd
kubectl create namespace argocd
helm template argocd argo-cd-2.2.3.tgz --namespace=argocd --values=values.yaml --values=secrets.yaml --include-crds | kubectl apply --namespace=argocd --filename=-
kubectl --namespace=argocd wait deployment/argocd-server --for=condition=Available --timeout=300s
kubectl --namespace=argocd port-forward service/argocd-server 8080:80 &
```

Open a browser at the location http://localhost:8080/, ignore the invalid certificate warning (Argo CD comes with an self-signed certificate unless you customize it) and log in using `admin` as username and `password` as password (of course you would customize it in a real setup).

## Deploying the application

Run the following commands in the terminal (choose which version of Helm you want to use):

- if you want to deploy the application using Helm 3:

```sh
kubectl apply --namespace=argocd --filename=hello-world.yaml
```

- else if you want to deploy the application using Helm 2:

```sh
kubectl apply --namespace=argocd --filename=hello-world-legacy.yaml
```

Then go to Argo CD web interface, click on the `sync` button of the hello-world application and wait for it to sync before executing the next commands in the terminal:

```sh
kubectl --namespace=hello-world wait deployment/hello-world --for=condition=Available --timeout=300s
kubectl --namespace=hello-world port-forward service/hello-world 8081:80 &
```

Open a browser at the location http://localhost:8081/ and note the decrypted secret ;-)
