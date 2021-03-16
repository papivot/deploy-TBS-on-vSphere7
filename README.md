# Deploy Tanzu Build Service on vSphere 7 with Tanzu

1. Harbor is enabled on the Supervisor cluster (NSX networking).
2. For this example, Harbor is running on `https://192.168.10.164`. 
3. A guest cluster has been deployed using the yaml provided in this repo. This is important as there are additional storage that needs to be carved out on the worker nodes. 
4. For this example, the guest cluster is deployed in `demo1` namespace in the Supervisor cluster 
5. Pod security policy has been relaxed in the guest cluster using the yaml provided in this repo. 

```shell
docker login https://192.168.10.164
```

```shell
$ kbld relocate -f /tmp/images.lock --lock-output /tmp/images-relocated.lock --repository 192.168.10.164/demo1/build-service --registry-ca-cert-path /tmp/harbor.crt
```

```shell
$ ytt -f /tmp/values.yaml -f /tmp/manifests/ -f /tmp/harbor.crt -v docker_repository="192.168.10.164/demo1/build-service" -v docker_username="administrator@vsphere.local" -v docker_password='Password' | kbld -f /tmp/images-relocated.lock -f- | kapp deploy -a tanzu-build-service -f- -y
```
