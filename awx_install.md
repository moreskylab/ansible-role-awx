

Install Ansible AWS



## Kubernetes with AWX Operator

### Step 1: Install Kubernetes (K3s)

```bash
# Install K3s (lightweight Kubernetes)
curl -sfL https://get.k3s.io | sh -

# Configure kubectl
sudo cp /etc/rancher/k3s/k3s.yaml ~/.kube/config
sudo chown $USER:$USER /etc/rancher/k3s/k3s.yaml
sudo chown $USER:$USER ~/.kube/config
export KUBECONFIG=~/.kube/config

# Verify installation
sudo kubectl get nodes
```

### Step 2: Install AWX Operator

```bash
export AWX_OPERATOR_TAG=2.19.0
git clone https://github.com/ansible/awx-operator.git
cd awx-operator
git tag
git checkout tags/$AWX_OPERATOR_TAG
```

### manually create a file called `kustomization.yaml` with the following content:

```bash
cat > kustomization.yaml << EOF
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  # Find the latest tag here: https://github.com/ansible/awx-operator/releases
  - github.com/ansible/awx-operator/config/default?ref=$AWX_OPERATOR_TAG
#  - awx-demo.yml

# Set the image tags to match the git version from above
images:
  - name: quay.io/ansible/awx-operator
    newTag: $AWX_OPERATOR_TAG
# Specify a custom namespace in which to install AWX
namespace: awx
EOF
```

Apply the operator settings

```bash
kubectl config set-context --current --namespace=awx
kubectl apply -k .
```

NOTE: Wait a bit and you should have the `awx-operator` running:

```bash
$ kubectl get pods -n awx
NAME                                               READY   STATUS    RESTARTS   AGE
awx-operator-controller-manager-66ccd8f997-rhd4z   2/2     Running   0          11s


```

Next, create a file named `awx-demo.yml` in the same folder with the suggested content below. The `metadata.name` you provide will be the name of the resulting AWX deployment.

```bash
cat > awx-demo.yaml << EOF
---
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx-demo
spec:
  service_type: nodeport
  nodeport_port: 32000
namespace: awx
EOF
```



Retrive password

```bash
kubectl get secret awx-demo-admin-password -n awx -o jsonpath="{.data.password}" | base64 --decode ; echo
```



Configuring a Let's Encrypt SSL certificate on Kubernetes for AWX typically involves using `cert-manager` to automate the certificate issuance and renewal process.

1. Install cert-manager:

Add the Jetstack Helm repository.

```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
```

install cert-manager using helm.

```bash
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.18.2 \ # Use the latest stable version
  --set installCRDs=true
```

2. Create a ClusterIssuer (or Issuer):
- Create a YAML file (e.g., `letsencrypt-clusterissuer.yaml`) for a `ClusterIssuer` to represent the Let's Encrypt ACME server and define the challenge method (e.g., HTTP-01 or DNS-01).

```yaml
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
	server: https://acme-v02.api.letsencrypt.org/directory
	email: your-email@example.com # Replace with your email
	privateKeySecretRef:
	  name: letsencrypt-prod-account-key
	solvers:
	  - http01:
		  ingress:
			class: nginx # Or your ingress controller class name
```

Apply the ClusterIssuer.

```bash
kubectl apply -f letsencrypt-clusterissuer.yaml
```

3. Configure Ingress for AWX:
- Modify the AWX Ingress resource (or create one if it doesn't exist) to include the `cert-manager.io/cluster-issuer` annotation and the `tls` section, referencing the secret that will store the certificate.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: awx-ingress
  annotations:
	cert-manager.io/cluster-issuer: letsencrypt-prod # Or your Issuer name
spec:
  tls:
	- hosts:
		- awx.your-domain.com # Replace with your AWX domain
	  secretName: awx-tls-secret # Secret where the certificate will be stored
  rules:
	- host: awx.your-domain.com # Replace with your AWX domain
	  http:
		paths:
		  - path: /
			pathType: Prefix
			backend:
			  service:
				name: awx-service # Or your AWX service name
				port:
				  number: 80
```

Apply the Ingress resource.

```bash
kubectl apply -f awx-ingress.yaml
```

REF:-

[Basic Install - Ansible AWX Operator Documentation](https://ansible.readthedocs.io/projects/awx-operator/en/latest/installation/basic-install.html)

YT:- [Install Ansible AWX with K3s on Ubuntu 22.04/24.04 | Ep. 5 - YouTube](https://www.youtube.com/watch?v=lW5CZlCCWyM)

[Ansible AWX Installation on Ubuntu 22.04 / 24.04 (https://youtu.be/lW5CZlCCWyM) · GitHub](https://gist.github.com/edmondgyampoh/76da2f6b64e4c42ed426224d4d7709de)


