# OpenWhisk Deployment for Mesos

[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0)
[![Build Status](https://travis-ci.org/apache/incubator-openwhisk-deploy-mesos.svg?branch=master)](https://travis-ci.org/apache/incubator-openwhisk-deploy-mesos)

This repository is part of [Apache OpenWhisk](http://openwhisk.incubator.apache.org/) and can be used to deploy OpenWhisk to a Mesos cluster.

## Subprojects

* TODO

## Travis builds

Each tool in this repository has to provide travis build scripts inside a `.travis` folder.
The folder should define 2 scripts:
* `setup.sh` - invoked during `before_install` phase
* `build.sh` - invokes during `script` phase
