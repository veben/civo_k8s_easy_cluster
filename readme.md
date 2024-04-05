# Civo k8s easy cluster

## I. Installation
### 1. Civo CLI
> See: https://www.civo.com/docs/overview/civo-cli
- Install the Civo CLI
```sh
curl -sL https://civo.com/get | sh
sudo mv /tmp/civo /usr/local/bin/civo
```
- Define the API key
> API key has to be recovered from Civo IHM
```sh
export CIVO_API_KEY=**********
civo apikey add civo-key ${CIVO_API_KEY}
civo apikey current civo-key
```
- Define the region
```sh
civo region ls
civo region use FRA1
```

### 2. K9s
> See: https://k9scli.io/topics/install/

### 3. kubectl
> See: See: https://kubernetes.io/docs/tasks/tools/install-kubectl/

## II. Creation of the cluster
- Define cluster parameters as environment variables
```sh
CLUSTER_NAME="civo-easy-cluster"
CLUSTER_NODES=1
CLUSTER_SIZE="g4s.kube.xsmall"
CLUSTER_REGION="FRA1"
```
- Create the cluster
> We have to wait around 2 minutes
```sh
civo kubernetes create ${CLUSTER_NAME} \
--size=${CLUSTER_SIZE} \
--nodes=${CLUSTER_NODES} \
--region=${CLUSTER_REGION} \
--wait
```
> The newly created cluster can be vizualize on Civo IHM: https://dashboard.civo.com/kubernetes
- Get some information about the cluster
```sh
civo --region=${CLUSTER_REGION} kubernetes show ${CLUSTER_NAME}
```

## III. Connection to the cluster with **K9s**
- Create a directory to store the configuration file to connect to the cluster
```sh
mkdir -p config
```
- Recover the config of the cluster and store it in `k3s.yaml` file
```sh
civo --region=${CLUSTER_REGION} \
kubernetes config ${CLUSTER_NAME} > ./config/k3s.yaml
```
- Configure `KUBECONFIG` environment variable to point on the config file
```sh
export KUBECONFIG=$PWD/config/k3s.yaml
```
- Launch k9s
```sh
k9s --all-namespaces
```

## IV. Deployment of first application
- Creation of the deployment
```sh
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.39 -- /agnhost netexec --http-port=8080
```
- Expose it via a service
```sh
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```
- Test the application
```sh
curl `k get service | grep hello-node | awk '{print $4}'`:8080
```
> I should send something like that:
> NOW: 2024-04-05 13:34:52.750022158 +0000 UTC m=+282.238384456%

## V. Deployment of Mario Bros
- Create the deployment
```sh
k apply -f k8s_mario/deployment.yaml
```
- Create the service
```sh
k apply -f k8s_mario/service.yaml
```
> Wait some second for the service to get an IP address
- Recover the URL:port
```sh
echo `k get service | grep hello-node | awk '{print $4}'`:80
```
- Copy the result to a browser to access to the game

## Delete the cluster
```sh
CLUSTER_NAME="civo-easy-cluster"
CLUSTER_REGION="FRA1"
civo kubernetes remove ${CLUSTER_NAME} --region=${CLUSTER_REGION} --yes
```
> The Kubernetes cluster (civo-easy-cluster) has been deleted