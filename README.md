
# Deploying Ollama on OpenShift

This guide walks you through deploying Ollama on OpenShift, including setting up a private registry, preparing Ollama and related images, and configuring OpenShift manifests.

## Prerequisites

- OpenShift Command Line Interface (CLI) installed.
- Podman and podman-compose installed.
- Access to an OpenShift cluster.

## Installation Steps

### Setting Up Gitea

1. Navigate to the Gitea directory and start Gitea using podman-compose:

```bash
cd gitea && podman-compose up -d
```

### Installing a Private Registry with Mirror-Registry

1. **Download the mirror-registry binary and install packages**:

First, set variables for easy customization:

```bash
BASTIONFQDN=$(hostname -f) # Your bastion fully qualified domain name
MIRROR_REGISTRY_VERSION="latest" # Change as required
INSTALL_DIR="/opt/mirror-registry"
```

Download and install the mirror-registry:

```bash
wget "https://developers.redhat.com/content-gateway/rest/mirror/pub/openshift-v4/clients/mirror-registry/${MIRROR_REGISTRY_VERSION}/mirror-registry.tar.gz" -O "${INSTALL_DIR}/mirror-registry.tar.gz"
cd "${INSTALL_DIR}"
tar zxvf mirror-registry.tar.gz
./mirror-registry install --initPassword=12345678 --targetHostname $BASTIONFQDN
```

2. **Add Quay rootCA certificate to trust store**

For RPM flavored machines:

```bash
cp $quayRoot/quay-rootCA/rootCA.pem /etc/pki/ca-trust/source/anchors/

update-ca-trust extract
```

For DEB flavored machines:

```bash
cp ~/quay-install/quay-rootCA/rootCA.pem /usr/local/share/ca-certificates/rootCA.crt
sudo update-ca-certificates
```

3. **Add Quay CA to the OCP cluster**:

```bash
oc create configmap quay-ca --from-file=${BASTIONFQDN}:=./quay-enterprise.pem -n openshift-config
oc patch image.config.openshift.io/cluster --patch '{"spec":{"additionalTrustedCA":{"name":"quay-ca"}}}' --type=merge
```

4. **Test authentication against the private Quay repository**:

```bash
podman login -u init -p 12345678 ${BASTIONFQDN}:8443
# Expected output: "Login Succeeded!"
```

### Preparing Ollama and Open-WebUI Images for OCP

1. **Build images using provided Dockerfiles**:

```bash
cd dockerfiles/ollama-ocp && podman build -t ollama-ocp:1.0 .
cd ../ollama-web-ocp && podman build -t open-web-ocp:1 .
```

2. **Tag and push images to the private registry**:



```bash
podman tag ollama-ocp:1.0 ${BASTIONFQDN}:8443/init/ollama-ocp:1.0
podman push ${BASTIONFQDN}:8443/init/ollama-ocp:1.0
podman tag open-web-ocp:1 ${BASTIONFQDN}:8443/init/open-web-ocp:1.0
podman push ${BASTIONFQDN}:8443/init/open-web-ocp:1.0
```

### Preparing OCP Manifests

1. **Modify `ollama-deployment.yaml` for GPU usage (if applicable)**:

```yaml
spec:
  containers:
  - name: ollama
    resources:
      limits:
        nvidia.com/gpu: "1"
```

2. **Apply the manifests**:

```bash
oc apply -f ./ocp
```

3. **Monitor the deployment**:

```bash
oc get all
```

### Testing Ollama GUI and API

1. **Pull models using the API**:

Replace `ollama-service.apps.lab.local` with your service's domain name.

```bash
curl -k https://ollama-service.apps.lab.local/api/pull -d '{"name": "gemma:2b"}'
curl -k https://ollama-service.apps.lab.local/api/pull -d '{"name": "phi"}'
```
2. **Submit a prompt**:

```bash
curl -k https://ollama-service.apps.lab.local/api/generate -d '{
     "model": "llama2",
     "prompt": "Explain complex numbers"
   }'
 ```
```bash
curl -k https://ollama-service.apps.lab.local/api/chat -d '{
  "model": "mistral",
  "messages": [
    { "role": "user", "content": "explain quaternions" }
  ]
}'
```

3. Use Open WebUI
