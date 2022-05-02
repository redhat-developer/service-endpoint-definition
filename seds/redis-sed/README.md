This helm chart defines a Redis Service Endpoint Definition (SED). When the SED chart is installed it provides the user with the opportunity to provide binding data as well as credentials to authenticate. The following values are to be customized while installing the SED chart:

1. Hostname
2. Password

The SED chart will render a secret with the binding data. This secret is compliant with the Service Binding Specification’s [Well Known Secret Entries](https://github.com/servicebinding/spec#well-known-secret-entries). Therefore, the secret rendered by the Redis SED chart is a bindable service endpoint that can be projected to workloads using the Service Binding specification’s [Binding Direct Secret Reference](https://github.com/servicebinding/spec#direct-secret-reference).