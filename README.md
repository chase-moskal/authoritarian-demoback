
# chase's workshop backend demo

- kubernetes yaml definitions describe how microservices fit together to form the app backend
- dev config is committed
- prod config is saved as github secrets, pushed during continuous deployment

## how to azure kubernetes

1. create the cluster in the azure portal ui
	- keep everything in the same location (uswest or w/e)
	- create a resource group
	- create a kubernetes cluster in that group
		- ensure `rbac` is disabled
		- ensure `http-application-routing` is disabled
1. create static ip's
	- ingress: follow cli instructions to create a static ip for your resource group
		```sh
		# get the full group name
		az aks show --resource-group demogroup --name workback2 --query nodeResourceGroup -o tsv

		# create static ip, use group and cluster names
		az network public-ip create \
			--resource-group MC_demogroup_workback2_westus \
			--name workback2-out-ip \
			--allocation-method static
		```
	- egress: aks automatically allocates a static egress ip by default (check this in the azure portal ui)
	- ensure the ip's are set to "Standard" sku instead of "Basic"
1. helm install nginx-ingress into the cluster
	- run the following command but with the correct ingress static ip you created
		```
		helm install nginx-ingress stable/nginx-ingress \
			--set rbac.create=false \
			--namespace ingress-basic \
			--set controller.replicaCount=2 \
			--set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
			--set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
			--set controller.service.loadBalancerIP="13.83.101.38"
		```
1. create an ingress.yml
	- with annotation `kubernetes.io/ingress.class: nginx`
	- map your subdomains to your services
1. helm install your application to deploy it
	- edit your `/etc/hosts` aliasing your ingress ip to your subdomains
	- test the application locally
	- remove edits from `/etc/hosts` when you're done testing
1. setup an `A` record on your domain pointing at the ingress ip
	- `A` record like `backend.mydomain.com` to the ip
	- also an `A` record for wildcard, like `*.backend.mydomain.com`

## higher level tutorial

prerequisites

- have your docker images ready and pushed to dockerhub

instructions

1. create a helm app, test on minikube
	- install and start minikube
	- create helm repo, setup templates, values, and config secrets
	- deploy on minikube and test
2. setup an azure kubernetes cluster
	- create cluster in azure portal
	- reserve static ip addresses for ingress and egress
	- set `A` records on your domain pointing to the ingress ip
	- install nginx-ingress controller into cluster
	- install acme cert-manager into cluster
3. deploy app to cluster
	- using helm install

## next topics to integrate into above tutorials

- github actions for continuous delivery
