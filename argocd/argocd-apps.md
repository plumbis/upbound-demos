# Upbound GitOps

## Prerequisites

- `git` installed locally.
- A [GitHub](https://www.github.com) GitHub account.
- [Rancher Desktop](#rancher-desktop) installed locally.
- An [AWS Account](#aws-keys).
- An [Upbound Cloud account](#upbound-cloud).

### Fork the repo
**Tasks:**
1. Fork this repo.
2. Create a REPO_URL environmental variable.

Using [GitHub](https://github.com/upbound/demos/tree/main/argocd), fork this repo to your personal GitHub space.

HTTPS clone the repo to your local environment.

**NOTE:** You *must* use HTTPS to clone the repo for this demo. Do not use the SSH clone method.

After cloning, set the REPO_URL environmental variable.  
`export REPO_URL=$(git config --get remote.origin.url)`

<details>
<summary>I can't use HTTPS to clone the repo</summary>
If you cannot use HTTPS to clone the repo you will still need to use the HTTPS GitHub URL.

![HTTPS URL from GitHub](images/https-repo-url.png)

Use this HTTPS address to create the `REPO_URL` environmental variable. From the above example image  
`export REPO_URL=https://github.com/plumbis/upbound-demos.git`
</details>

**Verify:**
```bash
echo $REPO_URL
```

<details>
<summary>Example Output</summary>

```bash
echo $REPO_URL
https://github.com/plumbis/upbound-demos.git
```

</details>

### Rancher Desktop
This demo relies on [Rancher Desktop](https://rancherdesktop.io/) to provide the Kubernetes infrastructure. Any other Kubernetes environment can be used, but an ingress must be used to access the ArgoCD dashboard.  

<details>
<summary> I'm not using Rancher Desktop</summary>
If another environment is used replace the localhost address `127.0.0.1` used in the ArgoCD Helm command with your external IP.

```bash
helm upgrade --install argocd argo/argo-cd --namespace argocd --create-namespace --set server.ingress.hosts="{argo-cd.127.0.0.1.nip.io}" --values argocd/helm-values.yaml --wait
```

</details>

### AWS Keys
**Tasks:**
1. Generate AWS Access and Secret Keys  
2. Register both keys as environmental variables  
3. Create an `aws-creds.conf` file containing both keys  

AWS is leveraged to create an cloud thingy. You will need an `AWS Access Key` and an `AWS Secret Key`. Instructions for creating your keys are available in the [AWS Documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html)

Create local variables `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`

```bash
export AWS_ACCESS_KEY_ID="AKIAIOSFODNN7EXAMPLE"
export AWS_SECRET_ACCESS_KEY="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

Create a text file named `aws-creds.conf` that will be used to generate a Kubernetes secret resource

```bash
echo "[default]
aws_access_key_id = $AWS_ACCESS_KEY_ID
aws_secret_access_key = $AWS_SECRET_ACCESS_KEY
" >aws-creds.conf
```

**Verify:**
```bash
echo AWS Access Key: $AWS_ACCESS_KEY_ID
echo 
echo AWS Secret Key: $AWS_SECRET_ACCESS_KEY
echo 
echo "Credentials File:"
cat aws-creds.conf
```

<details>
<summary> Example Output</summary>

```bash
AWS Access Key: AKIAIOSFODNN7EXAMPLE

AWS Secret Key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

Credentials File:
[default]
aws_access_key_id = AKIAIOSFODNN7EXAMPLE
aws_secret_access_key = CwJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY  
```

</details>

### Upbound Cloud
**Tasks:**
1. Create an Upbound Cloud account.
2. Create a Hosted control plane.
3. Create an Upbound user token.
4. Save the user token as an environmental variable.
5. Install the `up` CLI on your local system.
6. Login to Upbound Cloud with `up login`
7. Save your control plane ID as an environmental variable.

Upbound provides a fully managed control plane for things.  

Follow the [documentation](https://cloud.upbound.io/docs/getting-started/create-account) to create a free account on the [Upbound Cloud](https://cloud.upbound.io). 

Once your account is created and you login to the cloud administration portal, create a new `Hosted`.

Next, generate a [user token](https://cloud.upbound.io/docs/upbound-cloud/connecting-to-control-planes) in the Upbound Cloud. Register this token as a variable as the variable `UP_TOKEN` on your local system.

Replace the sample token below with your private token.  

```bash
export UP_TOKEN="u7TWaSb5P77g2yGGCWYH9856cy3ePCe5bMXywnBuaFXFSr3QqjsKj45KnwSt4byKLZwmnp4mdGqMJQvtCdM4nD8WcLxJekkPXRt7bymRk6wTRERdwdeJJZ7MS9NcZ....6bdF2vvuq"
```

Install the `up` CLI on your local system according to the [Upbound Documentation](https://cloud.upbound.io/docs/cli).

Login to your Upbound Cloud account using the `up login` command.

Verify your login with the `up ctp list` command to see your configured Upbound managed control plane.

```bash
up ctp list
NAME           ID                                     SELF-HOSTED   STATUS
demo   f1bea9c1-ef28-4f84-abcd-1234ba095bd   false         ready
```

Use the `ID` value to create the local variable `UP_CTP_ID`.  

`export UP_CTP_ID="f1bea9c1-ef28-4f84-abcd-1234ba095bd"`

**Verify:**

```bash
echo Control Plane ID: $UP_CTP_ID
echo 
echo User Token: $UP_TOKEN
```

<details>
<summary>Example Output</summary>

```bash
Control Plane ID: f1bea9c1-ef28-4f84-abcd-1234ba095bd

User Token: u7TWaSb5P77g2yGGCWYH9856cy3ePCe5bMXywnBuaFXFSr3QqjsKj45KnwSt4byKLZwmnp4mdGqMJQvtCdM4nD8WcLxJekkPXRt7bymRk6wTRERdwdeJJZ7MS9NcZ.uZcYUdpxTyttyNhE2zGCGt9Qz43hfXrq2tpLqd2NCMg87bBrRSQcvaXxqzyEQtnp9ChH-PjBTjtLwr5Et8exwfqFwgGSWHk9_j8L9c6tAn5a5jQRx999puwghn8DJZCMcaZ3jAqngKMFECyTX3aqGtRT8Ts53yDU4ebVbHNtXZnVE9hCnwVp4MgnBeu9ynkcr5kfwrsuuJeNgJLTnQQdJj4TybhMNmZSDNCYYQVV9EqqjhyWnCJzr5avhQm4FSuNPdHdq8c2B7te9HrNnCf3apUAEuuzcGZGV7u9UcxnCdZsX79ESmJmDTx3WUyQcBVhFSAe2vzkej6bdF2vvuq
```

</details>

## Configure Upbound Cloud
**Tasks:**
1. Generate a kubeconfig file.
2. Enable the Upbound providers.
3. Generate a Kubernetes secret object with your AWS credentials.

### Generate a kubeconfig file 
Generate a *kubeconfig* file with the settings for your Upbound Cloud control plane.  
This simplifies the `kubectl` command settings for the rest of this demo.  

`up ctp kubeconfig get $UP_CTP_ID --token=$UP_TOKEN --file kubeconfig-up-apps.yaml`

The settings are written to `kubeconfig-up-apps.yaml`

**Verify**

```bash
cat kubeconfig-up-apps.yaml
```

<details>
<summary>Example Output</summary>

```yaml
apiVersion: v1
clusters:
- cluster:
    server: https://proxy.upbound.io/controlPlanes/f1bea9c1-ef28-4f84-abcd-1234ba095bd
  name: upbound-f1bea9c1-ef28-4f84-abcd-1234ba095bd
contexts:
- context:
    cluster: upbound-f1bea9c1-ef28-4f84-abcd-1234ba095bd
    user: upbound-f1bea9c1-ef28-4f84-abcd-1234ba095bd
  name: upbound-f1bea9c1-ef28-4f84-abcd-1234ba095bd
current-context: upbound-f1bea9c1-ef28-4f84-abcd-1234ba095bd
kind: Config
preferences: {}
users:
- name: upbound-f1bea9c1-ef28-4f84-abcd-1234ba095bd
  user:
    token: u7TWaSb5P77g2yGGCWYH9856cy3ePCe5bMXywnBuaFXFSr3QqjsKj45KnwSt4byKLZwmnp4mdGqMJQvtCdM4nD8WcLxJekkPXRt7bymRk6wTRERdwdeJJZ7MS9NcZ.uZcYUdpxTyttyNhE2zGCGt9Qz43hfXrq2tpLqd2NCMg87bBrRSQcvaXxqzyEQtnp9ChH-PjBTjtLwr5Et8exwfqFwgGSWHk9_j8L9c6tAn5a5jQRx999puwghn8DJZCMcaZ3jAqngKMFECyTX3aqGtRT8Ts53yDU4ebVbHNtXZnVE9hCnwVp4MgnBeu9ynkcr5kfwrsuuJeNgJLTnQQdJj4TybhMNmZSDNCYYQVV9EqqjhyWnCJzr5avhQm4FSuNPdHdq8c2B7te9HrNnCf3apUAEuuzcGZGV7u9UcxnCdZsX79ESmJmDTx3WUyQcBVhFSAe2vzkej6bdF2vvuq
```

</details>

### Enable the Upbound Providers
Using the `kubeconfig-up-apps.yaml` configuration file add providers to your Upbound Cloud control plane.

- AWS:  
`kubectl --kubeconfig kubeconfig-up-apps.yaml apply --filename crossplane-config/provider-aws.yaml`
- SQL:  
`kubectl --kubeconfig kubeconfig-up-apps.yaml apply --filename crossplane-config/config-sql.yaml`
- Custom App:  
`kubectl --kubeconfig kubeconfig-up-apps.yaml apply --filename crossplane-config/config-app.yaml`

**Verify:**
```bash
kubectl --kubeconfig kubeconfig-up-apps.yaml get providers
```

<details>
<summary>Example Output</summary>

```bash
kubectl --kubeconfig kubeconfig-up-apps.yaml get providers
NAME                             INSTALLED   HEALTHY   PACKAGE                                 AGE
crossplane-provider-aws          True        True      crossplane/provider-aws:v0.24.1         21h
crossplane-provider-helm         True        True      crossplane/provider-helm:v0.10.0        21h
crossplane-provider-kubernetes   True        True      crossplane/provider-kubernetes:v0.3.0   21h
crossplane-provider-sql          True        True      crossplane/provider-sql:v0.4.1          21h
```
</details>

### Generate and Use the AWS Secret Object
Use your AWS credentials to generate a secret object in the `upbound-system` namespace  
`kubectl --kubeconfig kubeconfig-up-apps.yaml --namespace upbound-system create secret generic aws-creds --from-file creds=./aws-creds.conf`

Next, use this token to generate an AWS provider configuration  
`kubectl --kubeconfig kubeconfig-up-apps.yaml apply --filename crossplane-config/provider-config-aws-up.yaml`


**Verify:**
```bash
kubectl --kubeconfig kubeconfig-up-apps.yaml get secrets aws-creds -n upbound-system  

kubectl --kubeconfig kubeconfig-up-apps.yaml describe providerconfigs.aws.crossplane.io default
```

<details>
<summary>Example Output</summary>

```yaml
kubectl --kubeconfig kubeconfig-up-apps.yaml get secrets aws-creds -n upbound-system
NAME        TYPE     DATA   AGE
aws-creds   Opaque   1      21h

kubectl --kubeconfig kubeconfig-up-apps.yaml describe providerconfigs.aws.crossplane.io default
Name:         default
Namespace:
Labels:       <none>
Annotations:  <none>
API Version:  aws.crossplane.io/v1beta1
Kind:         ProviderConfig
Metadata:
  Creation Timestamp:  2022-05-11T20:48:48Z
  Finalizers:
    in-use.crossplane.io
  Generation:  1
  Managed Fields:
    API Version:  aws.crossplane.io/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:finalizers:
          .:
          v:"in-use.crossplane.io":
      f:status:
    Manager:      crossplane-aws-provider
    Operation:    Update
    Time:         2022-05-11T20:48:48Z
    API Version:  aws.crossplane.io/v1beta1
    Fields Type:  FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .:
          f:kubectl.kubernetes.io/last-applied-configuration:
      f:spec:
        .:
        f:credentials:
          .:
          f:secretRef:
            .:
            f:key:
            f:name:
            f:namespace:
          f:source:
    Manager:         kubectl-client-side-apply
    Operation:       Update
    Time:            2022-05-11T20:48:48Z
  Resource Version:  623737527
  UID:               a4fd6381-c410-4a92-a653-f869b655eaa5
Spec:
  Credentials:
    Secret Ref:
      Key:        creds
      Name:       aws-creds
      Namespace:  upbound-system
    Source:       Secret
Status:
Events:  <none>
```

</details>

## Install and Configure ArgoCD

### Install ArgoCD
**Tasks:**
1. Add the Helm repo
2. Update the Helm cache
3. Install the ArgoCD Helm chart
4. Generate an ArgoCD Kubernetes secret


Install ArgoCD on your Kubernetes environment
- Add the Helm Repo  
`helm repo add argo https://argoproj.github.io/argo-helm`
- Update Helm  
` helm repo update`
- Install the ArgoCD Helm Chart  
`helm upgrade --install argocd argo/argo-cd --namespace argocd --create-namespace --set server.ingress.hosts="{argo-cd.127.0.0.1.nip.io}" --values argocd/helm-values.yaml --wait`

Next, use the following code snippet to generate a Kubernetes secret in the argocd namespace.  

```yaml
echo "apiVersion: v1
kind: Secret
metadata:
  name: upbound
  labels:
    argocd.argoproj.io/secret-type: cluster
type: Opaque
stringData:
  name: upbound
  server: $SERVER
  config: |
    {
      \"bearerToken\": \"$UP_TOKEN\"
    }" \
    | kubectl \
    --namespace argocd \
    apply --filename -
```

**Verify:**
```bash
kubectl get pods -n argocd
echo 
kubectl describe secret upbound -n argocd
```

<details>
<summary>Example Output</summary>

```bash
kubectl get pods -n argocd
NAME                                                READY   STATUS    RESTARTS   AGE
argocd-redis-6484cccf5-rm4pk                        1/1     Running   0          15h
argocd-applicationset-controller-77dffc56b4-sfnqj   1/1     Running   0          15h
argocd-notifications-controller-667b6f8674-q6l2k    1/1     Running   0          15h
argocd-dex-server-7847bfd667-vhjrw                  1/1     Running   0          15h
argocd-server-d588c9ccd-hvbdm                       1/1     Running   0          15h
argocd-repo-server-787b6699dc-dws6d                 1/1     Running   0          15h
argocd-application-controller-0                     1/1     Running   0          15h

kubectl describe secret upbound -n argocd
Name:         upbound
Namespace:    argocd
Labels:       argocd.argoproj.io/secret-type=cluster
Annotations:  <none>

Type:  Opaque

Data
====
name:    7 bytes
server:  75 bytes
config:  490 bytes
```

</details>

Now you should be able to access the ArgoCD web interface at using the default credentials:  
**URL** [http://argo-cd.127.0.0.1.nip.io](http://argo-cd.127.0.0.1.nip.io)  
**Username** `admin`  
**Password** `admin123`  
  
<details>
<summary> ArgoCD Login</summary>

![ArgoCD Login](images/argo-login.gif)

</details>

### Generate ArgoCD Kubernetes Custom Resources
**Tasks:**
1. Create an AppProject CRD
2. Customize and create two Application CRDs

Using the YAML file in this forked repo, create an ArgoCD `AppProject` Kubernetes CRD named `upbound`.
`kubectl --namespace argocd apply --filename argocd/project.yaml`

Next, customize the provided Application manifests to use your Upbound control plane URL and git repo.

Confirm that both the `$SERVER` and `$REPO_URL` environmental variables have been set.

```bash
echo $SERVER
echo $REPO_URL
```

Edit both the `argocd/app-infra.yaml` and `argocd/app-apps.yaml` files, replacing `spec.source.repoURL` with your GitHub repo HTTPS URL and `spec.destination.server` with your Upbound control plane URL.

The following sed commands will edit both files, generating two new files named `customized-app-infra.yaml` and `customized-app-apps.yaml`. 

```bash
cat argocd/app-infra.yaml \
    | sed -e "s@server: .*@server: $SERVER@g" \
    | sed -e "s@repoURL: .*@repoURL: $REPO_URL@g" \
    | tee argocd/customized-app-infra.yaml

cat argocd/app-apps.yaml \
    | sed -e "s@server: .*@server: $SERVER@g" \
    | sed -e "s@repoURL: .*@repoURL: $REPO_URL@g" \
    | tee argocd/customized-app-apps.yaml
```

Create the `Application` CRDs by applying the new manifests.  
`kubectl --namespace argocd apply --filename argocd/customized-app-infra.yaml`  
`kubectl --namespace argocd apply --filename argocd/customized-app-apps.yaml`


**Verify:**
```bash
kubectl get appproject upbound -n argocd
kubectl get application upbound-infra -n argocd
kubectl get application upbound-apps -n argocd
```

<details>
<summary>Sample Output</summary>

```bash
kubectl get appproject upbound -n argocd
NAME      AGE
upbound   5h51m

kubectl get application upbound-infra -n argocd
NAME            SYNC STATUS   HEALTH STATUS
upbound-infra   Synced        Healthy

kubectl get application upbound-apps -n argocd
NAME           SYNC STATUS   HEALTH STATUS
upbound-apps   Synced        Healthy
```

</details>