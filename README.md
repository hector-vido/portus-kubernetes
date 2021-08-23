# Portus on Kubernetes

This repository configure Portus on Kubernetes.

Inside `portus.yml` there is a pod with two containers, one of them is responsible for the web dashboard, another one, with the same image, is responsible for repositories updates through a background task.

Portus uses a separated registry, like [registry v2](https://hub.docker.com/_/registry/), and acts like an frontend/authenticator for this registry. All interactions from command line, like `docker login` or `docker push` should be done directly with the registry.

When the registry is registered in Portus, remember to add an `External Registry Name` like `registry.192-168-99-104.nip.io`.

## Deploying

This repository was tested with minikube using [nip.io](https://nip.io/) as hostname.

First, make shure to obtain minikube ip to update all necessary files:

```bash
IP=$(minikube ip | tr -s . -) # 192-168-99-104
echo portus/portus.yml registry/registry.yml registry/config.yml | xargs -n1 sed -i "s/@IP@/$IP/g"
```

Second, create the TLS certificates. This is importante because Kubernetes ingress enforce a match between the hostname and the CN in the certificate:

```bash
cd certs
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout portus.key -out portus.cert -subj "/CN=portus.$IP.nip.io"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout registry.key -out registry.cert -subj "/CN=registry.$IP.nip.io"
```

Third, create the namespace and necessary objects:

```bash
kubectl create -f namespace.yml
kubectl create secret tls portus-cert --key certs/portus.key --cert certs/portus.cert -n registry
kubectl create secret tls registry-cert --key certs/registry.key --cert certs/registry.cert -n registry
kubectl create cm registry-conf --from-file registry/config.yml -n registry
kubectl create secret generic mysql-env --from-env-file mysql/mysql.env -n registry
```

Fourth, create the portus and registry deployments and the database statefulset:

```bash
echo mysql/mysql.yml portus/portus.yml registry/registry.yml | xargs -n1 kubectl create -f
```

# Docker

Remember to add this "insecure registry" inside `/etc/docker/daemon.json`:

```
{
  "insecure-registries": ["registry.192-168-99-104.nip.io"]
}
```
