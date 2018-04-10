DCOS_ENDPOINT ?= <DC/OS URL>
APIGATEWAY_DNS ?= <gateway endpoint>
OPENWHISK_UNIVERSE ?= <Universe URL>
WSK_NAMESPACE ?= guest
WSK_AUTH_TOKEN ?= <enter token here>

cli:
	curl -s https://downloads.dcos.io/binaries/cli/darwin/x86-64/dcos-1.10/dcos -o dcos
	chmod +x dcos
	./dcos --version
	./dcos cluster setup $(DCOS_ENDPOINT)

repo:
	./dcos package repo add --index=0 openwhisk-universe $(OPENWHISK_UNIVERSE)

.PHONY: openwhisk-install
openwhisk-install: exhibitor-install kafka-install invoker-install controller-install test-helloworld

.PHONY: openwhisk-uninstall
openwhisk-uninstall: invoker-uninstall controller-uninstall kafka-uninstall exhibitor-uninstall

.PHONY: apigateway-install
apigateway-install:
	yes | ./dcos package install apigateway

.PHONY: exhibitor-install
exhibitor-install:
	yes | ./dcos package install exhibitor
	echo "$$(tput setaf 4)waiting until the exhibitor is up (serving) ...$$(tput sgr0)"
	until (curl -s http://exhibitor-dcos.$(APIGATEWAY_DNS)/exhibitor/v1/cluster/status | jq . | grep "serving" | wc -l | grep 3); do printf '.'; sleep 5; done
	echo "$$(tput setaf 2)exhibitor is up!$$(tput sgr0)"

.PHONY: kafka-install
kafka-install:
	if (curl -s http://exhibitor-dcos.$(APIGATEWAY_DNS)/exhibitor/v1/cluster/status | jq . | grep "serving" | wc -l | grep 3); then \
		echo "$$(tput setaf 4)exhibitor is running. installing kafka ...$$(tput sgr0)"; \
		yes | ./dcos package install kafka --options=config/kafka.json; \
		until ((./dcos kafka update status --name=kafka | grep -v COMPLETE) || echo "OK") | grep "OK"; do printf '.'; sleep 5; done; \
		echo "$$(tput setaf 2)kafka is up!$$(tput sgr0)"; \
	else \
		echo "$$(tput setaf 1)exhibitor is not running. Cancelling kafka installation.$$(tput sgr0)"; \
	fi

.PHONY: couchdb-install
couchdb-install:
	echo "$$(tput setaf 4)installing whisk-couchdb ...$$(tput sgr0)"
	yes | ./dcos package install whisk-couchdb
	until (curl -s -I http://whisk-couchdb.$(APIGATEWAY_DNS) | grep "HTTP/1.1 200 OK"); do printf '.'; sleep 5; done
	echo "$$(tput setaf 2)couchdb is up!$$(tput sgr0)"

.PHONY: invoker-install
invoker-install:
	if ((./dcos kafka update status --name=kafka | grep -v COMPLETE) || echo "OK") | grep "OK"; then \
		echo "$$(tput setaf 4)installing invoker ...$$(tput sgr0)"; \
		yes | ./dcos package install whisk-invoker; \
		until (curl -s http://whisk-invoker.$(APIGATEWAY_DNS)/ping | grep "pong"); do printf '.'; sleep 5; done; \
		echo "$$(tput setaf 2)invoker is up!$$(tput sgr0)"; \
	else \
		echo "$$(tput setaf 1)kafka is not running. Cancelling invoker installation.$$(tput sgr0)"; \
	fi

.PHONY: controller-install
controller-install:
	if ((./dcos kafka update status --name=kafka | grep -v COMPLETE) || echo "OK") | grep "OK"; then \
		echo "$$(tput setaf 4)installing controller ...$$(tput sgr0)"; \
		yes | ./dcos package install whisk-controller; \
		until (curl -s http://whisk-controller.$(APIGATEWAY_DNS)/ping | grep "pong"); do printf '.'; sleep 5; done; \
		echo "$$(tput setaf 2)controller is up!$$(tput sgr0)"; \
		until (curl -s http://whisk-controller.$(APIGATEWAY_DNS)/invokers | grep "up"); do printf '.'; sleep 5; done; \
		echo "$$(tput setaf 2)at least one invoker is healthy!$$(tput sgr0)"; \
	else \
		echo "$$(tput setaf 1)kafka is not running. Cancelling controller installation.$$(tput sgr0)"; \
	fi

.PHONY: exhibitor-uninstall
exhibitor-uninstall:
	echo "$$(tput setaf 4)uninstalling exhibitor ...$$(tput sgr0)"
	printf "exhibitor-dcos\n" | ./dcos package uninstall exhibitor --app-id=exhibitor-dcos
	until (curl -s http://exhibitor-dcos.$(APIGATEWAY_DNS)/exhibitor/v1/cluster/status | grep -v "serving"); do printf '.'; sleep 5; done
	echo "$$(tput setaf 2)exhibitor was successfully uninstalled!$$(tput sgr0)"

.PHONY: kafka-uninstall
kafka-uninstall:
	echo "$$(tput setaf 4)uninstalling kafka ...$$(tput sgr0)"
	yes | ./dcos package install kafka --cli --global
	printf "kafka\n" | ./dcos package uninstall kafka --app-id=kafka
	# need to sleep for some time (20 sec) before kafka enters the shutdown-in-progress phase
	sleep 20
	until (./dcos kafka update status --name=kafka | grep "Could not reach the service scheduler"); do printf '.'; sleep 5; done
	echo "$$(tput setaf 2)kafka was successfully uninstalled!$$(tput sgr0)"

# only for <= DC/OS 1.9
kafka-cleanup:
	docker run mesosphere/janitor /janitor.py -r kafka-role -p dcos-kafka -z dcos-kervice-kafka \
		-m $(DCOS_ENDPOINT)/mesos/master/ \
		-n $(DCOS_ENDPOINT)/service/marathon/v2/apps/ \
		-e $(DCOS_ENDPOINT)/exhibitor/ \
		-a $(shell ./dcos config show core.dcos_acs_token)

.PHONY: controller-uninstall
controller-uninstall:
	echo "$$(tput setaf 4)uninstalling controller ...$$(tput sgr0)"
	printf "whisk-controller\n" | ./dcos package uninstall whisk-controller
	until (curl -s http://whisk-controller.$(APIGATEWAY_DNS)/ping | grep -v "pong"); do printf '.'; sleep 5; done
	echo "$$(tput setaf 2)controller was successfully uninstalled!$$(tput sgr0)"

.PHONY: invoker-uninstall
invoker-uninstall:
	echo "$$(tput setaf 4)uninstalling invoker ...$$(tput sgr0)"
	printf "whisk-invoker\n" | ./dcos package uninstall whisk-invoker
	until (curl -s http://whisk-invoker.$(APIGATEWAY_DNS)/ping | grep -v "pong"); do printf '.'; sleep 5; done
	until (curl -s http://whisk-controller.$(APIGATEWAY_DNS)/invokers | grep -v "up"); do printf '.'; sleep 5; done
	echo "$$(tput setaf 2)invoker was successfully uninstalled!$$(tput sgr0)"

.PHONY: test-helloworld
test-helloworld:
	echo "testing hello-world action"
	echo "$$(tput setaf 4)creating hello-world action ...$$(tput sgr0)"
	curl -s -u "$(WSK_AUTH_TOKEN)" http://whisk-controller.$(APIGATEWAY_DNS)/api/v1/namespaces/$(WSK_NAMESPACE)/actions/hello-world -XPUT -d '{"namespace":"$(WSK_NAMESPACE)","name":"hello-world","exec":{"kind":"nodejs:6","code":"function main(){return {\"body\": \"Hello, World!\"};}"}}' -H "Content-Type: application/json"
	echo "$$(tput setaf 4)invoking hello-world action ...$$(tput sgr0)"
	if (curl -s -u "$(WSK_AUTH_TOKEN)" "http://whisk-controller.$(APIGATEWAY_DNS)/api/v1/namespaces/$(WSK_NAMESPACE)/actions/hello-world?blocking=true&result=true" -XPOST) | grep "Hello, World!"; then \
		echo "$$(tput setaf 2)successfully invoked hello-world action$$(tput sgr0)"; \
	else \
		echo "$$(tput setaf 1)can not invoke hello-world action$$(tput sgr0)"; \
	fi
	echo "$$(tput setaf 1)deleting hello-world action ...$$(tput sgr0)"
	curl -s -u "$(WSK_AUTH_TOKEN)" http://whisk-controller.$(APIGATEWAY_DNS)/api/v1/namespaces/$(WSK_NAMESPACE)/actions/hello-world -XDELETE
