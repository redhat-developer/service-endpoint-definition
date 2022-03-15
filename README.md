# Service Endpoint Definition
Repository to share service endpoint definitions (SEDs) 

Kubernetes applications often depend on one or more cloud services. Services could be running on local cluster, but also, services could be running on remote clusters and public cloud providers. The introduction of [Service Binding Operator](https://github.com/redhat-developer/service-binding-operator) (SBO) kick started an effort to standarized the way OpenShift applications bind with cloud services. SBO implements the [Service Binding Spec](https://github.com/servicebinding/spec#service-binding-specification-for-kubernetes) which defines the standard for both service providers and workloads/applications developers to follow to make it easy for applications to connect to services. The end goal is for each service provider to develop Kubernetes controllers that defined the abstraction of their services in kubernetes via CRDs that define their Service Instance Types (SIT). According to the service binding spec, these SITs are spec compliant if they provide a reference to a secret which contains the Service Endpoing Definition (SED). Onces the SIT make a reference to a SED, developer of application will have a standard way to read the SED data into their application. When a SIT makes a reference to the SED as per the spec, it is said that the controller implements Provisioned Services.

OpenShift SBO 1.0 was released end of 2021 and the Service Binding Spec 1.0 was released at the begining of 2022. Although there has been some early efforts to support the concept of Provisioned Services. The addoption of Service Binding spec is still in bootstrap stage. So, there are still multiple service providers that have not yet implemented the Provisioned Service concept leaving application developer wondering how they can start implementing the standard on their side. In order to cover that transition the service binding spec also introduces the concept of direct secret binding, which allows SEDs to be created manually by an administrator. Developer could already start implementing their applications to read data from these SEDs and be ready for the automation that will be available in the near future as service providers implement the service binding spec.

# SEDs as helm charts
This project will deliver SEDs as helm charts. The requirements of the helm charts are as follows:
1. Chart delivers a secret the follows the service binding spec.
1. Chart has a test that can verify the health of the SED being defined.

# Not a helm chart repository

The aim with this repo is not to create a helm chart reprository, but to create an environment for peer review of SEDs helm charts. Once the chart is developed it will be maintained here, but the releases of the chart will happend on helm chart repositories such as the [OpenShift Helm Chart Repository](https://github.com/openshift-helm-charts/charts). The OpenShift Helm Chart Repository is not only a helm repository it is also a Red Had helm chart certification flow that will verify your charts can run properly on OCP.

# How to use the SED chart to test your application can connect with target service

Once the SED chart is certified it should be published to the [Helm Chart Repository Index](https://charts.openshift.io/index.yaml). To use the chart add the repo to your helm repository:

```
helm repo add openshift-helm-charts https://charts.openshift.io/
```

Once repo is added, the sed chart will be available for install. For example the psql-sed chart is already certified. The following command can be run to verify the chart is available for psql-sed:

```
helm search repo openshift-helm-charts | grep psql-sed
openshift-helm-charts/redhat-psql-sed                   1.0.0           1.0.0           A Helm chart for PostgreSQL Service Endpoint De...
```

To install the chart, gather the information required by the chart to connect to the service. As an example, for the psql-sed chart you can run the following command to show the values expected by the chart at install time:

```
helm show values openshift-helm-charts/redhat-psql-sed
postgresql:
  sed:
    hostname: myhostname
    port: 5432
    username: myuser
    password: mypassword
    databasename: mydatabase
```

Once you gather the information for your service, create a local values.yaml file by running the following command:

```
helm show values openshift-helm-charts/redhat-psql-sed > values.yaml
```

Edit the local values.yaml file with the service information you gathered, and now you are ready to deploy the sed chart. For example, for psql-sed chart, install by calling the following command:

```
helm instal my-psql-sed openshift-helm-charts/redhat-psql-sed -f values.yaml
```

The SED can be tested by running the following command:

```
helm test my-psql-sed
```

For this psql-sed example, there should be a secret named io.servicebinding.my-psql-sed. Verify by checking your secrets:

```
oc get secrets | grep io.servicebinding
```

Since you have verified via helm test, your secret is ready for binding. To test create a ServiceBinding CR yaml file. In our example, the file is named psql-sbo.yaml:

```
cat psql-sbo.yaml 
apiVersion: servicebinding.io/v1alpha3
kind: ServiceBinding
metadata:
  name: psql-service-ssm-app

spec:
  workload:
    apiVersion: apps/v1
    kind:       Deployment
    name:       ssm-bee

  service:
    apiVersion: v1
    kind:       Secret
    name:       io.servicebinding.my-psql-sed
```

Assuming there is a Deployment named ssm-bee in your namepsace and [Service Binding Operator](https://redhat-developer.github.io/service-binding-operator/userguide/getting-started/installing-service-binding.html) is installed, SBO will project for you the secret's data to your Deployment after applying the above yaml:

```
oc apply -f psql-sbo.yaml
```

To verify the binding has occured verify the volumes and volume mounts section of your Deployment:

```
oc get deployment ssm-bee -o yaml
```

You can also verify the status of your ServiceBinding object:

```
oc get servicebinding psql-service-ssm-app -o yaml
```