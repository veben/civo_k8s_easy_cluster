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
> See: https://kubernetes.io/docs/tasks/tools/install-kubectl/

### 4. Helm
> See: https://helm.sh/docs/intro/install/

## II. Creation of the cluster
- Define the API key
> API key has to be recovered from Civo IHM
```sh
export CIVO_API_KEY=**********
civo apikey add civo-key ${CIVO_API_KEY}
civo apikey current civo-key
```
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

## III. Configuration of kubectl and k9s
- Create a directory to store the configuration file to connect to the cluster
```sh
mkdir -p config
```
- Recover the config of the cluster and store it in `config.yaml` file
```sh
civo --region=${CLUSTER_REGION} \
kubernetes config ${CLUSTER_NAME} > ./config/config.yaml
```
- Configure `KUBECONFIG` environment variable to point on the config file
```sh
export KUBECONFIG=$PWD/config/config.yaml
```
- Test **kubectl** and **k9s**
```sh
kubectl get pod
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
- Clone the Mario Bros chart
```sh
git clone git@github.com:veben/helm_chart_mario_bros.git
```
- Deploy the game
```sh
helm install mario-bros helm_chart_mario_bros
```

## Delete the cluster
```sh
CLUSTER_NAME="civo-easy-cluster"
CLUSTER_REGION="FRA1"
civo kubernetes remove ${CLUSTER_NAME} --region=${CLUSTER_REGION} --yes
```
> The Kubernetes cluster (civo-easy-cluster) has been deleted