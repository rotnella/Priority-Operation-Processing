Developer Kubernetes Setup
==========================
In order to have an Executor dynamically allocate Handler pods in Kubernetes, you will need your Kubernetes secret keys, and namespace.

Prerequisites
-------------
* A Kubernetes cluster! (minikube, etc.)
* Install [kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

Setup the Priority Operation Processing Namespace
---------------------------
Using 'kubectl' setup a new namespace. You may need to get permission from your kubernetes administrators to perform this action.

`kubectl create namespace pop`

The following command will default your context so you don't have to specify each time.

`kubectl config set-context $(kubectl config current-context) --namespace=pop`

Setup the Priority Operation Processing Service User
------------------------------

:warning: Please be sure you are on the right cluster and namespace!

```
cd deploy/resourcepool/kubernetes/setup
kubectl create -f pop-service-account.yaml
kubectl create -f RoleBinding.yaml
```
Confirm everything:
```
kubectl get serviceaccounts/pop-service
kubectl describe RoleBinding pop-edit-service
kubectl get serviceaccounts
```
You should see the _pop-service_ item

`kubectl get rolebindings`

You should see the _pop-edit-service_ role binding`

Create Credentials
------------------

:warning: Do **NOT** create/use credentials if you are working with a **Minikube** cluster. They are not necessary nor will they work!

Switch to your kubernetes instance and new pop context in kubernetes

:warning: Please replace `<your.k8s.instance>`

```
kubectl config use-context <your.k8s.instance>
kubectl config set-context <your.k8s.instance> --namespace=pop
```

Get the "secret name". Copy the result and use it as the "secret name" below.

`echo kubectl get sa -n pop pop-service -o jsonpath={.secrets[0].name}`

Create the cert for the "secret name"

`kubectl get secret <secret name> -n pop -o jsonpath='{.data.ca\.crt}' > pop-service.ca.cert`

Create the token for the "secret name". Note: On the windows/linux use -d instead of -D

`kubectl get secret <secret name> -n pop -o jsonpath='{.data.token}' | base64 -D > pop-service.sa.token`
