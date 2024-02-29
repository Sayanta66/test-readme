---
title: "Nirmata Kyverno Adapters"
description: ""
weight: "5"
---

Kyverno adapters extend and enrich policy decisions for integrations. The following adapters are used by Nirmata:
* [Nirmata Venafi Adapter](../nirmata-kyverno-adapters/nirmata-venafi-adapter/)
* [Nirmata CIS Adapter](../nirmata-kyverno-adapters/nirmata-cis-adapter/)

---
title: "Nirmata CIS Adapter"
description: ""
weight: "2"
---

The kube-bench adapter periodically runs a CIS-benchmark check using cron-job with a tool called kube-bench and produces a cluster-wide policy report, based on the Policy Report - Custom Resource Definition. For more information on installation and Helm values, see [kube-bench-adapter](https://github.com/nirmata/kyverno-charts/tree/main/charts/kube-bench-adapter).

---
title: "Nirmata Venafi Adapter"
description: ""
weight: "1"
---

The Nirmata Venafi Adapter helps extract public keys, certificates, or certchains stored securely in [Venafi CodeSign Protect](https://www.venafi.com/platform/codesign-protect). These are stored in configmap fields which in-turn can be referred to from Kyverno image-verify policies.

The Nirmata Venafi Adapter is available as a helm chart that can be installed on a Kubernetes cluster with Kyverno installed, using the helm package manager.

### Prerequisites

- **Helm**: Refer to the [official docs](https://helm.sh/docs/intro/install/) for installation.

### Installing the Nirmata Venafi Adapter chart

#### Adding the Kyverno Helm repository

The following commands add the Kyverno helm chart repository and update it accordingly:

````bash
helm repo add nirmata https://nirmata.github.io/kyverno-charts/
helm repo update nirmata
````

**(Optional)** If a custom CA (Certificate Authority) is used in the cluster, create a configmap corresponding to the same in the namespace using the cutom-ca.pem key:

````bash
kubectl -n kyverno create configmap <e.g. ca-store-cm> --from-file=custom-ca.pem=<cert file e.g. some-cert.pem>
````

#### Creating a namespace

Install the Venafi-Adapter in any namespace. The example uses `nirmata-venafi-adapter` as the namespace:

````bash
kubectl create namespace nirmata-venafi-adapter
````

#### Installing the Nirmata Venafi Adapter

The following command installs the Venafi-Adapter from nirmata helm repo in the `nirmata-venafi-adapter` namespace, with desired parameters:

````bash
helm install venafi-adapter nirmata/venafi-adapter --namespace nirmata-venafi-adapter --create-namespace
````

**(Optional)** Other parameters to the above command corressponding to custom CA, HTTP proxies, or NO_PROXY should be provided as needed:

````bash
--set customCAConfigMap=<e.g. ca-store-cm> --set systemCertPath=<e.g. /etc/ssl/certs> --set "extraEnvVars[0].name=HTTP_PROXY" --set "extraEnvVars[0].value=<e.g. http://test.com:8080>" ...
````

#### Checking the pods

Check if the pods are running successfully by running the below command:

````bash
kubectl -n nirmata-venafi-adapter get pods
````

#### Checking the CRD creation

The created CRD will be shown as an imagekey. Run the below command to check for the CRD:

````bash
kubectl get crd
````

### Testing a sample policy

To test a sample use-case, create an imagekey Custom Resource (CR). Verify that the imagekey fetches the venafi certificates or keys and saves it to a configmap. Next, create a policy that will help verify the acceptance or blocking of it as expected.

#### Creating the password secret

Before creating the imagekey, create a password secret for the Venafi environment using the following command:

````bash
kubectl create secret generic venafi-pwd-secret -n nirmata-venafi-adapter --from-literal password=<your-password> --as system:serviceaccount:nirmata-venafi-adapter:imagekey-controller
````

Replace `your-password` with the password of your choice.

**(Optional)** If needed, create an additional secret with the below command that will convey to Venafi that an additional X.509 cert has to be trusted:

````bash
kubectl create secret generic venafi-addl-cert-secret -n nirmata-venafi-adapter --from-file <addl-cert-key-filename>
````

Replace `addl-cert-key-filename` with the name of your certificate file. This additional secret is needed in some corner cases when enterprise customers use internal certificates.

#### Creating the CR yaml

Create an yaml file that will define the imagekey custom resource. The example uses `imagekey.yaml` as the name of the CR yaml. `imagekey.yaml` is shown below:

````bash
apiVersion: security.nirmata.io/v1
kind: ImageKey
metadata:
  name: imagekey1
  namespace: nirmata-venafi-adapter
spec:
  venafiPKCS11Config:
    authURL: "https://yourorg-tpp.se.venafi.com/vedauth"
    hsmURL: "https://yourorg-tpp.se.venafi.com/vedhsm"
    username: your-venafi-username
    passwordSecretName: <namespace>/<venafi-env1-password-secret>/<key-in-secret>
    label: <Environment-Label>
    interval: <in minutes>
    hostAlias: NA
    configMap: <namespace>/<some-configmap>/<confmap key matching that in policy>
    fetchType: <pubkey, certificate or certchain>
    imagepullsecret: <secretname for private registries>
    keyfetchClientRegistry: <"" or prefix before nirmata/ in uri, e.g. ghcr.io>
    additionalCertSecretName: <"" or e.g. default/addl-venafi-cert-secret/addl-cert-secret-key>
````

#### Applying the CR

Apply the Custom Resource in the namespace where the Venafi Adapter is installed with the following command:

````bash
kubectl -n nirmata-venafi-adapter create imagekey.yaml
````

#### Verifying the created CR

Check that the CR is created successfully with:

````bash
kubectl -n nirmata-venafi-adapter get imagekey
````

#### Running the first job

After verifying that the CR is successfully created, ensuring the running of the first job is essential. It will download the specified key to the configmap specified in the CR yaml file. The following command does that:

````bash
kubectl -n nirmata-venafi-adapter get cm <config map name in CR> -o yaml
````

Replace `config map name in CR` with the name of the configmap provided in the imagekey custom resource (CR).

#### Creating the imageverify policy

Create a Kyverno imageverify policy referring to the configmap field. Refer to the [Kyverno official documentation](https://kyverno.io/policies/other/verify-image/verify-image/) for the policy definition template.

#### Creating and checking the pods

After the policy creation, create a pod with the below command and check if it is getting blocked or allowed based on their signing with the Venafi keys:

````bash
kubectl run venafisignedpod --image=ghcr.io/some-user/some-image:signed-by-me
````

### Uninstalling the chart

The below command removes all the Kubernetes components associated with the Venafi Adapter chart and deletes the release:

````bash
helm -n <namespace> uninstall venafi-adapter
````

>Note: Replace `namespace` with the name of the namespace in which the chart is installed. The created CRD needs to be deleted manually.
