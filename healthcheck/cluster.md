# Cluster Healthcheck
Sources: [https://github.com/heptio/sonobuoy](https://github.com/heptio/sonobuoy)

## Prerequisites
Minimum kubernetes version is 1.10.0

## Installation
Checking the cluster requires to target a Kubernetes Master Node.

`root@master`
```
yum install -y golang git ~/.bash_profile
cat << EOF >>
export GOROOT=/root/go
export PATH=$GOROOT/bin:$PATH
EOF
. ~/.bash_profile
go get -u -v github.com/heptio/sonobuoy
```

## Run the check
You can now run the cluster healthcheck. The check sequence may take 1 hour and more.

`root@master`
```
sonobuoy run
```

Now... wait for it ends

`root@master`
```
sonobuoy status
while [ "$(sonobuoy status | grep running)" ] ; do
 sonobuoy status
 echo "..."
 sleep 60
done
```

## Check the results
Once check ended you need to check the logs and the results status.

`root@master`
```
sonobuoy logs
sonobuoy retrieve .
```

This will make an archive at your place. Unzip logs and and find the results.

Let's check the e2e logs.

`root@master`
```
tar -zxvf *_sonobuoy_*.tar.gz plugins/e2e/results/e2e.log
tail -n 5 plugins/e2e/results/e2e.log
SUCCESS! -- 161 Passed | 0 Failed | 0 Pending | 729 Skipped PASS
```

All the interpretations through snapshots are designed [here](https://github.com/heptio/sonobuoy/blob/master/docs/snapshot.md).
