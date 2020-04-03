# Argo CD example application using Helm Sops

## Requirements

To deploy this example application, you need a running Kubernetes cluster. You can get one using [Minikube](https://minikube.sigs.k8s.io/docs/start/).

You also need to have GnuPG, [Sops](https://github.com/mozilla/sops), [Helm](https://helm.sh/docs/intro/install/) and [Helm Sops](https://github.com/camptocamp/helm-sops) installed on your workstation.

Finally, clone this repository and open a terminal at its root.

## Using the example GPG keyring

There is an example GPG keyring in the `gpg-keyring` directory which contains the secret key of an administrator named John Doe (which you will impersonate) and the public key of the Argo CD instance which you're about to install. The Sops configuration file (`.sops.yaml`) in the repository references these GPG keys. Argo CD's private key is configured in the `argocd/secrets.yaml` file.

To make use of this GPG keyring, execute the following command in the terminal.

```sh
export GNUPGHOME="$(realpath gpg-keyring)"
```

Of course, in a real setup, you would generate a dedicated private key for Argo CD and use your own GPG key (or generate a new one) instead of John Doe's.

## Installing Argo CD

Run the following commands in the terminal (choose the Helm command to run according to the Helm version installed on your workstation):

```sh
cd argocd
kubectl create namespace argocd
# If you have Helm 3 installed on your workstation run
helm-sops template argocd argo-cd-1.8.3+c2c.1.tgz --namespace=argocd --values=values.yaml --values=secrets.yaml --include-crds | kubectl apply --namespace=argocd --filename=-
# else if you have Helm 2 installed on your workstation run
helm-sops template argo-cd-1.8.3+c2c.1.tgz --name=argocd --namespace=argocd --values=values.yaml --values=secrets.yaml | kubectl apply --namespace=argocd --filename=-
# endif
kubectl --namespace=argocd wait deployment/argocd-server --for=condition=Available --timeout=300s
kubectl --namespace=argocd port-forward service/argocd-server 8080:80 &
```

Open a browser at the location http://localhost:8080/, ignore the invalid certificate warning (Argo CD comes with an self-signed certificate unless you customize it) and log in using `admin` as username and `password` as password (of course you would customize it in a real setup).

## Deploying the application

Run the following commands in the terminal:

```sh
kubectl apply --namespace=argocd --filename=hello-world.yaml
# Wait for the application to sync in Argo CD UI before executing the next commands
kubectl --namespace=hello-world wait deployment/hello-world --for=condition=Available --timeout=300s
kubectl --namespace=hello-world port-forward service/hello-world 8081:80 &
```

Open a browser at the location http://localhost:8081/ and note the decrypted secret ;-)
