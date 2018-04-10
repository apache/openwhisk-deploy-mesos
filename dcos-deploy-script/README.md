Bash script for deploying OpenWhisk packages on DC/OS
=====================================================

# Set up
- Make sure the environment variables in `Makefile` are set correctly for your environment.
- Install DC/OS CLI: `make cli` (by default using v1.10)

# Install custom universes

In order to add OpenWhisk universe packages to the DC/OS cluster (if not yet done):

```bash
make repo
```

# Installation steps

## First installation on a fresh cluster

- `make apigateway-install`
- make sure Route53 record was set up correctly
- `make couchdb-install`

## Installation of OpenWhisk packages

```bash
make openwhisk-install
```

# Make commands deep dive
## Install APIGateway

```bash
make apigateway-install
```

## Install OpenWhisk packages

```bash
make openwhisk-install
```

This installs the OpenWhisk stack, including:
- Dedicated Zookeeper Exhibitor for OpenWhisk
- Kafka service
- OpenWhisk Controller
- OpenWhisk Invoker

There are also scripts for individual services, e.g. `kafka-install`.

## Uninstall OpenWhisk packages

```bash
make openwhisk-uninstall
```

There are also uninstalling scripts for individual services, e.g. `kafka-uninstall`.
