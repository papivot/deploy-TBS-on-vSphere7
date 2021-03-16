# Deploy Tanzu Build Service on vSphere 7 with Tanzu

## Install
1. Harbor is enabled on the Supervisor cluster (NSX networking).
2. For this example, Harbor is running on `https://192.168.10.164`. 
3. A guest cluster has been deployed using the yaml provided in this repo. This is important as there are additional storage that needs to be carved out on the worker nodes. 
4. For this example, the guest cluster is deployed in `demo1` namespace in the Supervisor cluster 
5. Pod security policy has been relaxed in the guest cluster using the yaml provided in this repo. 
6. Download the Harbor registry's SSL root cert (see image below) and move it the `/tmp/harbor.crt` folder of the workstation from which to run the setup - 

![alt text](https://github.com/papivot/deploy-TBS-on-vSphere7/blob/main/harbor.png?raw=true)

7. Follow the documentation - section `Prerequisites`
8. Follow the documentation - section `Installing` while leveraging some of the examples provided here - 

```shell
$ docker login https://192.168.10.164
```

```shell
$ kbld relocate -f /tmp/images.lock --lock-output /tmp/images-relocated.lock --repository 192.168.10.164/demo1/build-service --registry-ca-cert-path /tmp/harbor.crt
```

```shell
$ ytt -f /tmp/values.yaml -f /tmp/manifests/ -f /tmp/harbor.crt -v docker_repository="192.168.10.164/demo1/build-service" -v docker_username="administrator@vsphere.local" -v docker_password='Password' | kbld -f /tmp/images-relocated.lock -f- | kapp deploy -a tanzu-build-service -f- -y
```
This should complete the install of TBS.

---

## Install dependencies

9. Follow the documentation - section `Import Tanzu Build Service Dependencies` while leveraging some of the examples provided here - 

```shell
$ kp import -f /tmp/descriptor-100.0.69.yaml --registry-ca-cert-path /tmp/harbor.crt
```
---

## Build an application 

Example of building a OCI image using TBS for an appplciation on https://github.com/$GH_USERNAME/spring-petclinic

```cmd
$ export GH_USERNAME=your-git-repository
$ kp secret create my-git-cred --git-user $GH_USERNAME --git-url https://github.com
$ kp secret create my-registry-cred --registry 192.168.10.164/demo1 --registry-user administrator@vsphere.local
$ kp image  create spring-petclinic --tag 192.168.10.164/demo1/spring-petclinic --git https://github.com/$GH_USERNAME/spring-petclinic.git --git-revision main
$ kp image trigger spring-petclinic
$ kp build list
```
