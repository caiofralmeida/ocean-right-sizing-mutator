# Ocean Resource Mutator

Spot resource mutator is a server which intercepts API requests and validates resource requirements definition within the Pod Spec. This is done by configuring the [dynamic admission controller](https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/) to send webhook mutating requests to this server before the request is being persisted in K8s. The server integrates with your Spot account and validates the resource requirements defined by the user. If there is no definition, the resource will be mutated with the appropriate values from the Spot backend recommended by Ocean.

## Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [Run Locally](#run-locally)
- [Documentation](#documentation)
- [Getting Help](#getting-help)
- [Community](#community)
- [Contributing](#contributing)
- [License](#license)

## Requirements

- The target cluster must be integrated with [Ocean](https://spot.io/products/ocean/).
- The mutator requires a Spot account ID from the previously installed Ocean controller.

The deployment process will generate a Self Signed Certificate and will create a secret in your cluster for the resource mutator server

```bash
make gencerts-deploy
```

## Installation

To use the Ocean Right-Sizing Mutator, you need to perform the following 2 small changes:

1. Add the following `label: value` to any Namespace that you want the mutating webhook to intercept - `spot.io/mutate-resource: enabled`. Only Namespaces with this label will be processed as part of the mutating webhook configuration (see file - [mutatingwebhook-template.tmpl](./deployment/mutatingwebhook-template.tmpl))
2. Add the following label to your deployment file - `spot.io/mutate-resource: enabled`. The webhook server will only get tomutate deployment that contains this label (based on the `MutatingWebhookConfiguration` object configured while deploying the webhook). The webhook server will mutate the resource requests with recommendations from [Ocean Right-Sizing API](https://help.spot.io/spotinst-api/ocean/ocean-cloud-api/ocean-for-aws/get-right-sizing-recommendations/).

Here is an example of a deployment with the annotation above:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nodejs-backend
  labels:
    app: nodejs-backend
    spot.io/mutate-resource: enabled
  namespace: default
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs-backend
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nodejs-backend
    spec:
      containers:
        - image: tduek/nodejs-demo:latest
          imagePullPolicy: Always
          name: nodejs-backend
          ports:
            - containerPort: 3000
              protocol: TCP
          resources:
            requests:
              memory: "200Mi"
              cpu: "200m"
            limits:
              memory: "256Mi"
              cpu: "400m"
```

## Run Locally

To run the webhook server locally, create the certs for the server:

```bash
make gencerts-deploy
```

In addition to the certs above, the following environment variables should be set:

- SPOTINST_CONTROLLER_ID - the controller id for the Ocean cluster
- SPOTINST_TOKEN - token for Spot API
- SPOTINST_ACCOUNT - Spot account which the ocean cluster exists in

With the above environment variables set, run the server with:

```bash
make runlocal
```

## Documentation

For a comprehensive documentation, check out the [API documentation](https://help.spot.io/).

## Getting Help

We use GitHub issues for tracking bugs and feature requests. Please use these community resources for getting help:

- Ask a question on [Stack Overflow](https://stackoverflow.com/) and tag it with [ocean-right-sizing-mutator](https://stackoverflow.com/questions/tagged/ocean-right-sizing-mutator/).
- Join our Spotinst community on [Slack](http://slack.spot.io/).
- Open an [issue](https://github.com/spotinst/ocean-right-sizing-mutator/issues/new/choose/).

## Community

- [Slack](http://slack.spot.io/)
- [Twitter](https://twitter.com/spot_hq/)

## Contributing

Please see the [contribution guidelines](.github/CONTRIBUTING.md).

## License

Code is licensed under the [Apache License 2.0](LICENSE). See [NOTICE.md](NOTICE.md) for complete details, including software and third-party licenses and permissions.
