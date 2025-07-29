# Table of Contents

- [Crossplane](#crossplane)
- [UI](#ui)

# Crossplane
## Install Crossplane

```
cd crossplane-demo
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm upgrade -i crossplane --namespace crossplane-system --create-namespace crossplane-stable/crossplane
# Wait for crossplane to be installed before applying other resources
sleep 15
kubectl apply -f provider-aws.yaml
kubectl apply -f functions.yaml
k config set-context --namespace crossplane-system --current
```

## Provider Configuration
The ProviderConfig determines settings the Provider uses communicating to the external provider. Each Provider determines available settings of their ProviderConfig.

Provider authentication is usually configured with a ProviderConfig. For example, to use basic key-pair authentication with Provider AWS a ProviderConfig spec defines the credentials and that the Provider pod should look in the Kubernetes Secrets objects and use the key named aws-creds.

AWS Authentication keys example:

Create `aws-credentials.txt`
```
[default]
aws_access_key_id = <access_key>
aws_secret_access_key = <secret_key>
```

Create `aws-creds` secret
```
kubectl create secret generic \
aws-creds \
-n crossplane-system \
--from-file=secret-key=./aws-credentials.txt
```

Create providerConfig
```
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: user-keys
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: secret-key
```

```
apiVersion: s3.aws.upbound.io/v1beta1
kind: Bucket
metadata:
  name: <random_string>-user-bucket
spec:
  forProvider:
    region: us-east-1
  providerConfigRef:
    name: user-keys
```

## AWS Providers
- providers doc https://docs.upbound.io/providers
- crossplane doc https://docs.crossplane.io/latest/concepts/providers/
- provider-aws upbound https://marketplace.upbound.io/providers?query=aws
- provider-aws crossplane contrib https://github.com/crossplane-contrib/provider-aws
- provider-upjet-aws crossplane contrib https://github.com/crossplane-contrib/provider-upjet-aws

Provider is like install a kubernetes operator for each AWS resource with CRDs

### Provider upjet aws
https://github.com/crossplane-contrib/provider-upjet-aws

### Authentication
https://docs.upbound.io/providers/provider-aws/authentication


## Managed Resources
https://docs.crossplane.io/latest/concepts/managed-resources/

A managed resource (MR) represents an external service in a Provider. When users create a new managed resource, the Provider reacts by creating an external resource inside the Provider’s environment. Every external service managed by Crossplane maps to a managed resource.

- Amazon AWS EC2 Instance defined in provider-upjet-aws.
- Google Cloud GKE Cluster defined in provider-upjet-gcp.
- Microsoft Azure PostgreSQL Database defined in provider-upjet-azure.

### Functions
Installing a Function creates a function pod. Crossplane sends requests to this pod to ask it what resources to create when you create a composite resource.

```
apiVersion: pkg.crossplane.io/v1
kind: Function
metadata:
  name: function-patch-and-transform
spec:
  package: xpkg.crossplane.io/crossplane-contrib/function-patch-and-transform:v0.8.2
```

## Simulation
Preview changes https://docs.upbound.io/operate/simulations (IaC like `terraform plan`)

## Compositions, XRD, Claims

- Compositions - A template to define how to create resources. "Similar to terraform modules" (quoted).
- Composite Resource Definition (XRD) - A custom k8s API specification. "Similar to CRD, e.g. deployment" (quoted).
- Composite Resource (XR) - Created by using the custom API defined in a Composite Resource Definition. XRs use the Composition template to create new managed resources.
- Claims (XRC) - Like a Composite Resource, but with namespace scoping. Useful for end users, e.g. developers

### Compositions
https://docs.crossplane.io/latest/concepts/compositions/

Compositions are a template for creating multiple managed resources as a single object.

A Composition composes individual managed resources together into a larger, reusable, solution.

An example Composition may combine a virtual machine, storage resources and networking policies. A Composition template links all these individual resources together.


# UI
Komoplane from komodorio https://github.com/komodorio/komoplane

Can be installed into k8s. I will run `komoplane` from CLI with the k8s context for this demo

- Download binary from https://github.com/komodorio/komoplane/releases
- extract downloaded `gz`file
- set k8s context
- run binary `./komoplane`
- open browser `localhost:8090`