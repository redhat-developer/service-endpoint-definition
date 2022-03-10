# Service Endpoint Definition
Repository to share service endpoint definitions (SEDs) 

Kubernetes applications often depend on one or more cloud services. Services could be running on local cluster, but also, services could be running on remote clusters and public cloud providers. The introduction of [Service Binding Operator](https://github.com/redhat-developer/service-binding-operator) (SBO) kick started an effort to standarized the way OpenShift applications bind with cloud services. SBO implements the [Service Binding Spec](https://github.com/servicebinding/spec#service-binding-specification-for-kubernetes) which defines the standard for both service providers and workloads/applications developers to follow to make it easy for applications to connect to services. The end goal is for each service provider to develop Kubernetes controllers that defined the abstraction of their services in kubernetes via CRDs that define their Service Instance Types (SIT). According to the service binding spec, these SITs are spec compliant if they provide a reference to a secret which contains the Service Endpoing Definition (SED). Onces the SIT make a reference to a SED, developer of application will have a standard way to read the SED data into their application. When a SIT makes a reference to the SED as per the spec, it is said that the controller implements Provisioned Services.

OpenShift SBO 1.0 was released end of 2021 and the Service Binding Spec 1.0 was released at the begining of 2022. Although there has been some early efforts to support the concept of Provisioned Services. The addoption of Service Binding spec is still in bootstrap stage. So, there are still multiple service providers that have not yet implemented the Provisioned Service concept leaving application developer wondering how they can start implementing the standard on their side. In order to cover that transition the service binding spec also introduces the concept of direct secret binding, which allows SEDs to be created manually by an administrator. Developer could already start implementing their applications to read data from these SEDs and be ready for the automation that will be available in the near future as service providers implement the service binding spec.

# SEDs as helm charts
This project will deliver SEDs as helm charts. The requirements of the helm charts are as follows:
1. Chart delivers a secret the follows the service binding spec.
1. Chart has a test that can verify the health of the SED being defined.

# Not a helm chart repository

The aim with this repo is not to create a helm chart reprository, but to create an environment for peer review of SEDs helm charts. Once the chart is developmed it will be mantianed here, but the releases of the chart will happend on helm chart repositories such as the [OpenShift Helm Chart Reporitory](https://github.com/openshift-helm-charts/charts)
