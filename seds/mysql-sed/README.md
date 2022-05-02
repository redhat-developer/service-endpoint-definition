This Helm chart defines a MySQL Service Endpoint Definition (SED). When the SED chart is installed, it provides the user with the opportunity to provide binding data and credentials to authenticate. The following values must be customized while installing the SED chart:

1. Host
1. Port
1. Username
1. Password
1. Databasename

The SED chart will render a secret with the binding data. This secret is compliant with the Service Binding specification’s [Well Known Secret Entries](https://github.com/servicebinding/spec#well-known-secret-entries). Therefore, the secret rendered by the MySQL SED chart is a bindable service endpoint that can be projected to workloads using the Service Binding specification’s [Direct Secret Reference](https://github.com/servicebinding/spec#direct-secret-reference).