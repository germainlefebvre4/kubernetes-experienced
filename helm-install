# Helm Catalog Installation
Sources: https://github.com/helm/helm

This is a simple installation. Best ractice is to set a ServiceAccount with admin role to only allow privileged actions on Kubernetes.

## Installation
Purpose is to download and install Helm binary in a PATH directory.
`root@master`
```
wget https://storage.googleapis.com/kubernetes-helm/helm-v2.11.0-linux-amd64.tar.gz
tar -zxvf helm-v2.11.0-linux-amd64.tar.gz
mv linux-amd64 /etc/kubernetes/helm
sed -i 's#^\(PATH=.*\)#\1:/etc/kubernetes/helm#g' ~/.bash_profile
rm -f helm-v2.11.0-linux-amd64.tar.gz
. ~/.bash_profile
```

## Initialization
Simply initialize Helm on your system
`root@master`
```
helm init --upgrade
helm repo update
```

And add user `kube-system:default` in role `cluster-admin` to allow communicate through namespaces.
```
kubectl create clusterrolebinding add-on-cluster-admin --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

## Usage
You can look for Helm Packages.
`root@master`
```
helm search mysql
```
Output:
```
NAME                                    CHART VERSION   APP VERSION     DESCRIPTION
stable/mysql                            0.10.2          5.7.14          Fast, reliable, scalable, and easy to use open-source rel...
stable/mysqldump                        1.0.0           5.7.21          A Helm chart to help backup MySQL databases using mysqldump
stable/prometheus-mysql-exporter        0.2.1           v0.11.0         A Helm chart for prometheus mysql exporter with cloudsqlp...
stable/percona                          0.3.3           5.7.17          free, fully compatible, enhanced, open source drop-in rep...
stable/percona-xtradb-cluster           0.3.0           5.7.19          free, fully compatible, enhanced, open source drop-in rep...
stable/phpmyadmin                       1.3.0           4.8.3           phpMyAdmin is an mysql administration frontend
stable/gcloud-sqlproxy                  0.6.0           1.11            Google Cloud SQL Proxy
stable/mariadb                          5.2.3           10.1.37         Fast, reliable, scalable, and easy to use open-source rel...
```

And you can install it with the good package name (like stable/xxx)
`root@master`
```
helm install --name my-release stable/mysql
```

This will create a deployment prefix my-release
`root@master`
```
kubectl get deploy
```
Output:
```
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
my-release-mysql   1         1         1            0           2s
```

## Secure your Helm installation
Quick and dirty way to edit your actual configuration and limit actions to helm to admin-user which is not the defaut user.
```
kubectl -n kube-system edit deploy/tiller-deploy
```

Add these two lines in container (tiller) scope
```
serviceAccount: admin-user
serviceAccountName: admin-user
```

