
# Internal Registry Configuration in Openshift 4




## Exposing the Registry

- Openshift Internal registry is preconfigured during the cluster installation in OCP 4. However, it is not exposed. We can expose it by running below command.

- First check if route exists or not.

```bash
$ oc get route -n openshift-image-registry
No resources found in openshift-image-registry namespace. 
```
- As the route does not exist so lets create it.

```bash
$ oc patch configs.imageregistry.operator.openshift.io/cluster --patch '{"spec":{"defaultRoute":true}}' --type=merge
```

- Now check again.

```bash
$ oc get route -n openshift-image-registry
NAME            HOST/PORT                                                 PATH   SERVICES         PORT    TERMINATION   WILDCARD
default-route   default-route-openshift-image-registry.apps-crc.testing          image-registry   <all>   reencrypt     None
e
```
- Create a project in which you would be pushing the images.

```bash
$ oc new-project testing
```

## Accessing the Internal Openshift Registry on podman host

- Now try to login to the registry on a podman host. In following example podman is configured on the same node where we are running `oc` commands.


```bash
$ podman login -u kubeadmin -p `oc whoami -t` default-route-openshift-image-registry.apps-crc.testing
Login Succeeded!
```
- If you have podman on the different host and you dont have `oc` command on podman host so you can fetch the token by running `oc whoami -t`  from host where you have oc command installed and provide it in following command in place of token on the podman host.

```bash
$ podman login -u kubeadmin -p token default-route-openshift-image-registry.apps-crc.testing
Login Succeeded!
```

## Building/Pushing image to Openshift Internal Registry from podman host.

- Now pull or create docker image which you want to push to the Openshift Internal Registry. In following example I created an image from a `DockerFile` and tagged it properly to point to the Openshift Internal Registry.


```bash
$ podman build -f Dockerfile -t default-route-openshift-image-registry.apps-crc.testing/webimage:1
```

- Push the image to OCP registry.

```bash
$ podman push default-route-openshift-image-registry.apps-crc.testing/testingwebimage:01
```

- Check under image streams in testing project that new image is present.


```bash
$ oc get is -n testing
NAME     IMAGE REPOSITORY                                                         TAGS     UPDATED
webimage   default-route-openshift-image-registry.apps-crc.testing/testing/gymweb   1   1 minutes ago

```

## Now lets create pods/applications to use that image in the testing namespace

- First try to create a pod

```bash
$ oc run webs --image=default-route-openshift-image-registry.apps-crc.testing/testing/gymweb:1 -n testing

```

- Also try to create an application


```bash
$ oc new-app --name=webserver --image=default-route-openshift-image-registry.apps-crc.testing/testing/gymweb:01 -n testing
$ oc run webs --image=default-route-openshift-image-registry.apps-crc.testing/testing/gymweb:1 -n testing

```

## Allowing pods to reference images across projects, so that pods which are running in other projects can utilize images from testing project.

- To allow pods in project1 to reference images from testing project. After adding that role, the pods in project1 that reference the default service account would be able to pull images from testing project.

```bash
$ oc policy add-role-to-user system:image-puller system:serviceaccount:project1:default --namespace=testing

```

- To allow access for any service account in project1, use the group:

```bash
$ oc policy add-role-to-group system:image-puller system:serviceaccounts:project1 --namespace=testing

```
## Official OCP Documentation Links

[Exposing the registry](https://docs.openshift.com/container-platform/4.14/registry/securing-exposing-registry.html
)

[Allowing pods to reference images across projects](https://docs.openshift.com/container-platform/4.14/openshift_images/managing_images/using-image-pull-secrets.html)

## 🛠 Skills
Openshift, Kubernetes, Podman...


## Authors

- [@kaybee-singh](https://www.github.com/kaybee-singh)
