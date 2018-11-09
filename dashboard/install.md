# Kubernetes Dashboard

A Kubernetes Cluster has to be up and running before running next steps.
Sources: [https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)


## Installation
A quick and running Dashboard installation.

`root@master`
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```

## Access to Dashboard
### Connect to Dahsboard
Basically Kubernetes refuse any coonnection to Pods except the Cluster itself.
We need to temporarily open flow on a port.
`root@master`
```
kubectl proxy --address='euinkub10.cube-net.org' --port=8001 --accept-hosts='.*'
```

URL to access to Dashboard is weird because it is done through Proxy :
[http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/](http://localhost:8001/api/v1/namespaces/kube-system/services/https:kubernetes-dashboard:/proxy/)

### Sign-in on Dashboard
We have 2 options to open access to Dashboard : fastest and secure.

#### Secure option : Allow Service Accounts
We can allow access to non-privileged user (Service Accounts) to the Dashboard.

When user is created we just have to get the secret related to the user
`root@master`
```
kubectl -n kube-system get secret
kubectl -n kube-system describe secret kubernetes-dashboard-token-lfzpm
```

#### Fastest option : Allow OpenBar
We can temporarirly open thr Dashboard as admin to All-the-World.

`root@master`
```
cat << EOF > dashboard-admin.yaml
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
  labels:
    k8s-app: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: kubernetes-dashboard
  namespace: kube-system
EOF

kubectl apply -f dashboard-admin.yaml
```

Tadaam! Now we can access to the Dashboard as admin =)
