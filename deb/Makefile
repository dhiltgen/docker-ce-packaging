include ../containerd.mk

SHELL:=/bin/bash
ARCH:=$(shell uname -m)
ENGINE_DIR:=$(CURDIR)/../../engine
CLI_DIR:=$(CURDIR)/../../cli
GITCOMMIT?=$(shell cd $(CLI_DIR) && git rev-parse --short HEAD)
DOCKER_HUB_ORG?=dockereng
VERSION?=0.0.0-dev
GO_BASE_IMAGE=golang
GO_VERSION:=1.10.4
GO_IMAGE=$(GO_BASE_IMAGE):$(GO_VERSION)
SUFFIX=ce
DEB_VERSION?=$(shell SUFFIX=$(SUFFIX) ./gen-deb-ver $(CLI_DIR) "$(VERSION)")
CHOWN:=docker run --rm -v $(CURDIR):/v -w /v alpine chown
EPOCH?=4
DOCKER_IMAGE?=$(DOCKER_HUB_ORG)/$(ENGINE_IMAGE):$(VERSION)

ifdef BUILD_IMAGE
	BUILD_IMAGE_FLAG=--build-arg $(BUILD_IMAGE)
endif

COMMON_FILES=common
BUILD?=docker build \
	$(BUILD_IMAGE_FLAG) \
	--build-arg GO_IMAGE=$(GO_IMAGE) \
	--build-arg COMMON_FILES=$(COMMON_FILES) \
	--build-arg DOCKER_IMAGE=$(DOCKER_IMAGE) \
	--build-arg CONTAINERD_SHIM_PROCESS_IMAGE=$(CONTAINERD_SHIM_PROCESS_IMAGE) \
	-t debbuild-$@/$(ARCH) \
	-f $(CURDIR)/$@/Dockerfile .
RUN=docker run --rm -i \
	-e EPOCH='$(EPOCH)' \
	-e DEB_VERSION=$(word 1, $(DEB_VERSION)) \
	-e VERSION=$(word 2, $(DEB_VERSION)) \
	-e DOCKER_GITCOMMIT=$(GITCOMMIT) \
	-v $(CURDIR)/debbuild/$@:/build \
	-v $(CURDIR)/debbuild/docker:/docker \
	debbuild-$@/$(ARCH)

SOURCE_FILES=cli.tgz docker.service
SOURCES=$(addprefix sources/, $(SOURCE_FILES)) $(METADATA_FILE)
ENGINE_IMAGE=engine-community
ENGINE_SCOPE=ce
METADATA_FILE=common/distribution_based_engine.json

IMAGE_TAG=nightly

.PHONY: help
help: ## show make targets
	@awk 'BEGIN {FS = ":.*?## "} /^[a-zA-Z_-]+:.*?## / {sub("\\\\n",sprintf("\n%22c"," "), $$2);printf " \033[36m%-20s\033[0m  %s\n", $$1, $$2}' $(MAKEFILE_LIST)

.PHONY: clean
clean: ## remove build artifacts
	[ ! -d debbuild ] || $(CHOWN) -R $(shell id -u):$(shell id -g) debbuild
	$(RM) -r debbuild
	[ ! -d sources ] || $(CHOWN) -R $(shell id -u):$(shell id -g) sources
	$(RM) -r sources
	[ ! -d artifacts ] || $(CHOWN) -R $(shell id -u):$(shell id -g) artifacts
	$(RM) -r artifacts
	-docker rm docker2oci
	-rm -f $(METADATA_FILE)

../engine-$(ARCH)-docker-compat.tar:
	@echo 'You must run "make image" before attempting to build packages (or use CI pipelines)'
	/bin/false

$(METADATA_FILE): ../engine-$(ARCH)-docker-compat.tar
	docker load < $<
	docker inspect $(DOCKER_IMAGE) --format '{{ .Config.Labels.distribution_based_engine }}' > $@

.PHONY: deb
deb: ubuntu debian raspbian ## build all deb packages

.PHONY: ubuntu
ubuntu: ubuntu-bionic ubuntu-xenial ubuntu-trusty ## build all ubuntu deb packages

.PHONY: debian
debian: debian-stretch debian-jessie ## build all debian deb packages

.PHONY: raspbian
raspbian: raspbian-stretch debian-jessie ## build all raspbian deb packages

.PHONY: ubuntu-bionic
ubuntu-bionic: ## build ubuntu bionic deb packages
ubuntu-bionic: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: ubuntu-xenial
ubuntu-xenial: ## build ubuntu xenial deb packages
ubuntu-xenial: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: ubuntu-trusty
ubuntu-trusty: ## build ubuntu trusty deb packages
ubuntu-trusty: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: debian-buster
debian-buster: ## build debian buster deb packages
debian-buster: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: debian-jessie
debian-jessie: ## build debian jessie deb packages
debian-jessie: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: debian-stretch
debian-stretch: ## build debian stretch deb packages
debian-stretch: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: raspbian-jessie
raspbian-jessie: ## build raspbian jessie deb packages
raspbian-jessie: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

.PHONY: raspbian-stretch
raspbian-stretch: ## build raspbian stretch deb packages
raspbian-stretch: $(SOURCES)
	$(BUILD)
	$(RUN)
	$(CHOWN) -R $(shell id -u):$(shell id -g) debbuild/$@

sources/cli.tgz:
	mkdir -p $(@D)
	docker run --rm -i -w /v \
		-v $(CLI_DIR):/cli \
		-v $(CURDIR)/$(@D):/v \
		alpine \
		tar -C / -c -z -f /v/cli.tgz --exclude .git cli

sources/docker.service: ../systemd/docker.service
	mkdir -p $(@D)
	cp $< $@

docker.tgz:
	$(MAKE) -C ../static ENGINE_DIR=$(ENGINE_DIR) VERSION=$(VERSION) static-engine

# offline bundle
sources/engine.tar:
	$(MAKE) -C ../image ENGINE_IMAGE=$(ENGINE_IMAGE) ENGINE_SCOPE=$(ENGINE_SCOPE) engine-$(ARCH).tar
	mv ../image/engine-$(ARCH).tar $@
