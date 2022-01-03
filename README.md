# Kustomize Demo

This repository tries to collect the first steps with Kustomize that provides a solution for customizing Kubernetes resource configuration free from templates and DSLs.

Kustomize lets users customize raw, template-free YAML files for multiple purposes, leaving the original YAML untouched and usable as is.

Kustomize targets kubernetes; it understands and can patch kubernetes style API objects. It’s like make, in that what it does is declared in a file, and it’s like sed, in that it emits edited text.

## My first App

In this example, the first application will be deployed using Kustomize:

- Create required base elements

```$
mkdir myfirstapp
cd myfirstapp
vi deployment.yaml
vi service.yaml
vi route.yaml
```

- Create Kustomize objects in order to apply this objects in the namespace *myfirstapp* using a specific naming convenction prefixing with *myfirstapp-...*:

```$
kustomize create --resources deployment.yaml,service.yaml,route.yaml --namespace myfirstapp --nameprefix myfirstapp-
```

- Finally, it is time to build the final objects using Kustomize and apply them:

```$
oc new-project myfirstapp
kustomize build . | oc apply -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc apply -k ./
```

- Test the correct app deployment:

```$
ROUTE=$(oc -n myfirstapp get route myfirstapp-front-javascript-v1 -o jsonpath='{.spec.host}')
curl https://${ROUTE} -k | grep "React App"
...
    <title>React App</title>
```

In order to clean up the resources created, it is required to execute the following command:

```$
kustomize build . | oc delete -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc delete -k ./
```

## Secure App

In this example, an application with specific configuration will be deployed using Kustomize:

- Create required base elements, the deployment and the required configuration:

```$
mkdir secureapp
cd secureapp
vi deployment.yaml
vi application.properties
vi application-secret.properties
```

- Create Kustomize objects in order to apply this objects in the namespace *secureapp* using a specific naming convenction prefixing with *secureapp-...*:

```$
kustomize create --resources deployment.yaml --namespace secureapp --nameprefix secureapp-
```

- Add some custom configuration in the kustomization.yaml file:

```$
...
configMapGenerator:
- name: conf-plain
  files:
  - application.properties
secretGenerator:
- name: conf-secret
  files:
  - application-secret.properties
```

- Finally, it is time to build the final objects using Kustomize and apply them:

```$
oc new-project secureapp
kustomize build . | oc apply -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc apply -k ./
```

- Test the correct app deployment:

```$
PODID=$(oc get pod -l app=front-javascript -o jsonpath='{.items[0].metadata.name}' -n secureapp)
oc -n secureapp exec ${PODID} cat /etc/conf/application.properties
oc -n secureapp exec ${PODID} cat /etc/secret/application-secret.properties
```

In order to clean up the resources created, it is required to execute the following command:

```$
kustomize build . | oc delete -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc delete -k ./
```

## Image App

In this example, an application with specific image configuration will be deployed using Kustomize:

- Create the application deployment object:

```$
mkdir imageapp
cd imageapp
vi deployment.yaml
```

- Create Kustomize objects in order to apply this objects in the namespace *imageapp* using a specific naming convenction prefixing with *imageapp-...*:

```$
kustomize create --resources deployment.yaml --namespace imageapp --nameprefix imageapp-
```

- Add some custom configuration in the kustomization.yaml file:

```$
...
images:
- name: testing
  newName: quay.io/acidonpe/jump-app-front-javascript
  newTag: latest
```

- Finally, it is time to build the final objects using Kustomize and apply them:

```$
oc new-project imageapp
kustomize build . | oc apply -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc apply -k ./
```

- Test the correct app deployment:

```$
oc get po -n imageapp
NAME                                           READY   STATUS    RESTARTS   AGE
imageapp-front-javascript-v1-974cdd67c-h8t8z   1/1     Running   0          55s
```

In order to clean up the resources created, it is required to execute the following command:

```$
kustomize build . | oc delete -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc delete -k ./
```

## Label App

In this example, an application with specific labels and annotations will be deployed using Kustomize:

- Create the application deployment object:

```$
mkdir labelapp
cd labelapp
vi deployment.yaml
```

- Create Kustomize objects in order to apply this objects in the namespace *labelapp* using a specific naming convenction prefixing with *labelapp-...*:

```$
kustomize create --resources deployment.yaml --namespace labelapp --nameprefix labelapp-
```

- Add overlays configuration in order to include specific k8s objects:

```$
vi overlays/production/secret.yaml
vi overlays/staging/configmap.yaml
```

- Finally, it is time to build the final objects using Kustomize and apply them:

```$
oc new-project labelapp
kustomize build . | oc apply -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc apply -k ./
```

In order to clean up the resources created, it is required to execute the following command:

```$
kustomize build . | oc delete -f -

# Alternatively, it is possible to use oc client with Kustomize integration
oc delete -k ./
```

## Multi Envs App

In this example, multiple application will be deployed in multiple logical environments with specific parameters for each environment using Kustomize:

- Create the application deployment objects:

```$
mkdir -p multienvapp/base/front-javascript
mkdir -p multienvapp/base/back-golang
mkdir -p multienvapp/overlays/staging/
mkdir -p multienvapp/overlays/production/
cd multienvapp
vi multienvapp/base/front-javascript/deployment.yaml
vi multienvapp/base/front-javascript/service.yaml
vi multienvapp/base/front-javascript/route.yaml
vi multienvapp/base/back-golang/deployment.yaml
vi multienvapp/base/back-golang/service.yaml
vi multienvapp/base/back-golang/route.yaml
```

- Create Kustomize objects in order to apply this objects in the namespace *multienvapp* using a specific naming convenction prefixing with *multienvapp-...*:

```$
cd multienvapp/base/front-javascript
kustomize create --resources deployment.yaml,service.yaml,route.yaml
cd ../../..

cd multienvapp/base/back-golang
kustomize create --resources deployment.yaml,service.yaml,route.yaml
cd ../../..

cd multienvapp/overlays/staging/
kustomize create --resources ../../base/back-golang,../../base/front-javascript --namespace multienvapp-staging
cd ../../..

cd multienvapp/overlays/production
kustomize create --resources ../../base/back-golang,../../base/front-javascript --namespace multienvapp-production
cd ../../..
```

- Add some custom configuration in the kustomization.yaml files in production and staging folders:

```$
## PRODUCTION
commonLabels:
  version: v1
  env: production

## STAGING
commonLabels:
  version: v1
  env: staging
```

- Finally, it is time to build the final objects using Kustomize and apply them:

```$
oc new-project multienvapp-staging
kustomize build multienvapp/overlays/staging/ | oc apply -f -

oc new-project multienvapp-production
kustomize build multienvapp/overlays/production/ | oc apply -f -
```

- Test the correct app deployment:

```$
oc get pod -n multienvapp-production
oc get pod -n multienvapp-staging

```

In order to clean up the resources created, it is required to execute the following command:

```$
kustomize build multienvapp/overlays/staging/ | oc delete -f -
kustomize build multienvapp/overlays/production/ | oc delete -f -
```

## Author

Asier Cidon @RedHat