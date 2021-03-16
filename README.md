# Deploy Tanzu Build Service on vSphere 7 with Tanzu

```shell
kbld relocate -f /tmp/images.lock --lock-output /tmp/images-relocated.lock --repository 192.168.10.164/demo1/build-service --registry-ca-cert-path /tmp/harbor.crt
```

```shell
$ ytt -f /tmp/values.yaml -f /tmp/manifests/ -f /tmp/harbor.crt -v docker_repository="192.168.10.164/demo1/build-service" -v docker_username="administrator@vsphere.local" -v docker_password='Password' | kbld -f /tmp/images-relocated.lock -f- | kapp deploy -a tanzu-build-service -f- -y
```
