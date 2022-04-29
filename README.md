# Service Endpoint Definition
This is the repository to share service endpoint definitions (SEDs). SEDs are secrets containing Service Specific information such as endpoints and credentials. These secrets can be bound to running kubernetes workolads so that they are able to connect to the service they represent.

Kubernetes workloads often depend on one or more cloud services. While services could be running on the local cluster, they could also be running on remote clusters and public cloud providers. The introduction of [Service Binding Operator](https://github.com/redhat-developer/service-binding-operator) kick-started an effort to standardize the way OpenShift workloads bind with cloud services. Service Binding Operator implements the [Service Binding specification](https://github.com/servicebinding/spec#service-binding-specification-for-kubernetes) which defines the standard for both service providers and application developers to follow to easily connect workloads to services. The end goal is for each service provider to develop Kubernetes controllers that define the abstraction of their services in Kubernetes through custom resource definitions (CRDs) that define their Service Instance Types (SITs). According to the Service Binding specification, these SITs are spec-compliant if they provide a reference to a secret that contains the Service Endpoint Definition (SED). With spec-compliant SITs, application developers will have a standard way to read the SED data into their workload. When an SIT makes a reference to the SED as per the specification, the controller implements [Provisioned Service](https://github.com/servicebinding/spec#provisioned-service) method to enable services to be bindable.

Service Binding Operator 1.0 was released at the end of 2021 and the Service Binding Spec 1.0 was released at the beginning of 2022. Although there have been some early efforts to support the concept of Provisioned Service, the adoption of the Service Binding specification is still in the bootstrap stage. There are still multiple service providers that have not yet implemented the provisioned service concept leaving application developers wondering how they can start implementing the standard on their side. To cover that transition the Service Binding specification also introduces the concept of [direct secret reference](https://github.com/servicebinding/spec#direct-secret-reference), which allows SEDs to be created manually by an administrator. As service providers implement the Service Binding specification, developers can start implementing their workloads to read data from these SEDs and be ready for the automation that will be available in the near future.

With SEDs, an administrator can define bindable secrets that describe a service connection even if such services is not represented in the Kuberentes cluster. Examples of this type of services are services that are provisioned directly in public cloud providers. For service providers that want to implement Provisioned Services, the SEDs will serve as a guide for the type and format of data that workloads expect from their services when using Service Bindings. On the other hand, the application providers will be able to use SEDs to bind to services that are not even repersented with Kubernetes custome resources. 

# SEDs as helm charts
This project will deliver SEDs as Helm charts. The requirements of the Helm charts are as follows:
1. Chart delivers a secret that follows the Service Binding specification for a specific cloud service.
1. Chart has a test that can verify the health of the service represented by the SED being defined. See [Helm Chart Tests](https://helm.sh/docs/topics/chart_tests/) for more details and examples on how to implement a Helm test.


NOTE:

The aim of this repo is not to create a Helm chart repository, but to create an environment for peer review of SED Helm charts. After the chart is developed it will be maintained here, but the releases of the chart will happen on Helm chart repositories such as the [OpenShift Helm Chart Repository](https://github.com/openshift-helm-charts/charts). The OpenShift Helm Chart Repository is not only a Helm repository but also a Red Hat Helm chart certification flow that verifies whether your charts can run properly on the OpenShift Container Platform (OCP) clusters.

# Using the SED chart to connect your workload to the backing service

As a cluster administrator, you can use the SED chart to connect your workload to backing service.

Prerequisites:

- The SED chart is published on a Helm Chart Repository such as [Helm Chart Repository Index](https://charts.openshift.io/index.yaml)
- You have created a workload that can read data projected from the secrets as volumes.
- You have installed the Service Binding Operator from the OperatorHub.

## Procedure

1. To use the chart, add the Helm chart repository containing your SED Helm chart:

```
helm repo add openshift-helm-charts https://charts.openshift.io/
```

After the SED repository is added, the SED Helm chart is available for installation. For example, to verify that the psql-sed Helm chart is available for PostgreSQL SED, you can run the following command:

```
helm search repo openshift-helm-charts | grep psql-sed
```

**Example Output:**

```
openshift-helm-charts/redhat-psql-sed                   1.0.0           1.0.0           A Helm chart for PostgreSQL Service Endpoint Definition
```

The output verifies that the SED chart is now added and is available in the Helm chart repository.

2. To install the chart, gather the information required by the chart to connect to the service. For example, to gather service information for the psql-sed Helm chart  and show the values expected by the chart at the time of its installation, run the following command:

```
helm show values openshift-helm-charts/redhat-psql-sed
```

**Example Output:**

```
postgresql:
  sed:
    hostname: myhostname
    port: 5432
    username: myuser
    password: mypassword
    databasename: mydatabase
```

3. After you gather the information for your service, create a local `values.yaml` file:

```
helm show values openshift-helm-charts/redhat-psql-sed > values.yaml
```

4. Edit the local `values.yaml` file with the service information you gathered.

5. Deploy the PostgreSQL SED. For example, for the psql-sed Helm chart, run the following command:

```
helm install my-psql-sed openshift-helm-charts/redhat-psql-sed -f values.yaml
```

**Example Output:**

```
NAME: my-psql-sed
LAST DEPLOYED: Fri Apr 29 10:37:46 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
```

6. Verify that the secret got deployed: For example, there should be an `io.servicebinding.my-psql-sed` secret for the psql-sed Helm chart. You can verify the SED by checking your secrets:

```
oc get secrets | grep io.servicebinding
```

**Example Output:**

```
io.servicebinding.my-psql-sed                              servicebinding.io/postgresql          7      18m
```

7. Test the PostgreSQL SED deployed:

```
helm test my-psql-sed
```

**Example Output:**

```
NAME: my-psql-sed
LAST DEPLOYED: Fri Apr 29 10:37:46 2022
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE:     my-psql-sed-sed-test
Last Started:   Fri Apr 29 10:40:08 2022
Last Completed: Fri Apr 29 10:40:28 2022
Phase:          Succeeded
```

The previous output verifies that the service represented by the PostgreSQL SED is running as expected and bindable to any workload.


8. Create a `ServiceBinding` custom resource (CR) YAML file.

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

9. Apply the `Service Binding` CR to project the binding data of the secret to your `ssm-bee` deployment: 

```
oc apply -f psql-sbo.yaml
```

**Example Output:**

```
servicebinding.servicebinding.io/psql-service-ssm-app created
```

After applying the `psql-sbo` CR, assuming that there is already a `ssm-bee` deployment in your namespace and Service Binding Operator installed, Service Binding Operator projects the binding data to your deployment.

10. To verify that the binding has occurred, check the volumes and volume mounts sections under the `spec` field of your deployment. 

```
oc get deployment ssm-bee -o yaml
```

**Example Output:**

```
    spec:
      volumes:
        - name: psql-service-ssm-app
          secret:
            secretName: io.servicebinding.my-psql-sed
            defaultMode: 420
```

The output verifies that the io.servicebinding.my-psql-sed secret has been mounted and the binding has occurred.

11. Verify the status of your `ServiceBinding` instance. It shows that all conditions under status has the `status` field set to true:

```
oc get servicebinding.servicebinding.io psql-service-ssm-app -o yaml
```

**Example Output:**

```
status:
  binding:
    name: io.servicebinding.my-psql-sed
  conditions:
    - lastTransitionTime: '2022-05-02T18:52:10Z'
      message: ''
      reason: DataCollected
      status: 'True'
      type: CollectionReady
    - lastTransitionTime: '2022-05-02T18:52:10Z'
      message: ''
      reason: ApplicationUpdated
      status: 'True'
      type: InjectionReady
    - lastTransitionTime: '2022-05-02T18:52:10Z'
      message: ''
      reason: ApplicationsBound
      status: 'True'
      type: Ready
```

The output verifies that all conditions under the status field have their status field set to True. The output establishes that all the values from the secret are projected as binding data into the ssm-bee deployment and the workload is now connected to the backing service.