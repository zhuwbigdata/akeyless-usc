# k8s USC

## Create dedicated SA in k8s cluster

### Fixed Mode SA - token-request-sa 
```
kubectl apply -f fixed_mode_sa_token.yaml 
serviceaccount/token-request-sa unchanged
clusterrole.rbac.authorization.k8s.io/tokenrequest unchanged
clusterrolebinding.rbac.authorization.k8s.io/tokenrequest unchanged
secret/token-request-sa-token created

FIXED_SA_JWT_TOKEN=$(kubectl get secret token-request-sa-token -n waynez  \
  --output 'go-template={{.data.token | base64decode}}')
```

### Dynamic Mode SA - akeyless-jit 
```
kubectl apply -f dynamic_mode_sa_token.yaml 
serviceaccount/akeyless-jit created
clusterrole.rbac.authorization.k8s.io/sa-request created
clusterrolebinding.rbac.authorization.k8s.io/sa-request created
secret/akeyless-jit-token created

DYNAMIC_SA_JWT_TOKEN=$(kubectl get secret token-request-sa-token -n waynez \
  --output 'go-template={{.data.token | base64decode}}')
```

### Get k8s cluster details
```
CLUSTER_NAME='waynez-k8s-demo'
REGION='us-central1'
GKE_CLUSTER_CA_CERT=$(gcloud container clusters describe $CLUSTER_NAME --location $REGION --format="value(masterAuth.clusterCaCertificate)")
GKE_CLUSTER_ENDPOINT=$(gcloud container clusters describe $CLUSTER_NAME --region $REGION --format="value(endpoint)")
```

## Create generic k8s target
### CLI
```
akeyless target create k8s \
--name <Target name> \
--k8s-cluster-endpoint <K8S Cluster endpoint> \
--k8s-cluster-ca-cert <K8S Cluster certificate> \
--k8s-cluster-token <K8S Cluster authentication token>

```

### Create target with fixed mode SA
```
akeyless target create k8s \
--name /devops/k8s/fixed_mode_sa_target \
--k8s-cluster-endpoint https://${GKE_CLUSTER_ENDPOINT} \
--k8s-cluster-ca-cert ${GKE_CLUSTER_CA_CERT} \
--k8s-cluster-token ${FIXED_SA_JWT_TOKEN} \
--profile devopsapi
A new target named /devops/k8s/fixed_mode_sa_target was successfully created
```


### Create target with dynamic mode SA
```
akeyless target create k8s \
--name /devops/k8s/dynamic_mode_sa_target \
--k8s-cluster-endpoint https://${GKE_CLUSTER_ENDPOINT} \
--k8s-cluster-ca-cert ${GKE_CLUSTER_CA_CERT} \
--k8s-cluster-token ${DYNAMIC_SA_JWT_TOKEN} \
--profile devopsapi
A new target named /devops/k8s/dynamic_mode_sa_target was successfully created
```

## Create k8s USC
### CLI
```
akeyless create-usc -n <name> --target-to-associate <target name> --k8s-namespace <kubernerets namespace>
```

### Create USC with fixed mode SA targt
```
GW_URL=https://<your_gateway_url>
$ akeyless create-usc -n /devops/k8s/usc-k8s-fixed-sa -u ${GW_URL} --target-to-associate  /devops/k8s/fixed_mode_sa_target --k8s-namespace waynez --profile devopsapi
Universal Secrets Connector /devops/k8s/usc-k8s-fixed-sa successfully created
```

### Create USC with dynamc mode SA targt
```
GW_URL=https://<your_gateway_url>
$  akeyless create-usc -n /devops/k8s/usc-k8s-dynamic-sa -u ${GW_URL} --target-to-associate  /devops/k8s/dynamic_mode_sa_target --k8s-namespace waynez --profile devopsapi
Universal Secrets Connector /devops/k8s/usc-k8s-dynamic-sa successfully created
```