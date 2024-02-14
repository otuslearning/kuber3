# Instruction
## Before installation
Install minikube, start minikube, ingress addon should be disabled
## Add repositories
### Add helm repositories
* prometheus-community repository
    ```shell
    helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
    ```
* nginx-ingress repository
    ```shell
    helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
    ```
## Installation
### Install in the following order
1. install prometheus stack
    ```shell
    helm upgrade --install prometheus-stack -f prometheus/stack.values.yaml prometheus-community/kube-prometheus-stack --create-namespace -n monitoring
    ```
2. install ingress-nginx
    ```shell
    helm upgrade --install ingress-nginx -f nginx-ingress/values.yaml ingress-nginx/ingress-nginx --create-namespace -n ingress-nginx
    ```
    ```shell
    minikube addons enable ingress
    ```
3. install postgres database
    ```shell
    helm upgrade --install database -f postgres/values.yaml oci://registry-1.docker.io/bitnamicharts/postgresql  --create-namespace -n development
    ```
4. create database, user and password for application need install job
    ```shell
    helm install user-db ./user-db -n development
    ```
5. install user-service
    ```shell
    helm upgrade --install user-service ./user-service -n development
    ```
6. install postgres exporter
    ```shell
    helm upgrade --install postgres-exporter -f postgres-exporter/values.yaml prometheus-community/prometheus-postgres-exporter -n development
    ```
## Tests
Run jmeter 
```shell
./jmeter -n -t user-service-create.jmx
```
```shell
./jmeter -n -t user-service-read.jmx
```

## Uninstall
* uninstall postgres-exporter
```shell
helm uninstall postgres-exporter -n development
```
* uninstall user-service
```shell
helm uninstall user-service -n development
```
* delete job
```shell
helm uninstall user-db -n development
```
* uninstall postgres database
```shell
helm delete database -n development
```
```shell
#Delete PVC's associated with database
kubectl delete pvc data-database-postgresql-0  -n development
```
* uninstall prometheus stack
```shell
helm delete kube-prometheus-stack  -n monitoring
```
```shell
#Delete PVC's associated with prometheus-alertmanager
kubectl delete pvc storage-prometheus-alertmanager-0  -n monitoring
```

