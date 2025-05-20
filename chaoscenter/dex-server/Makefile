

dpl ?= deploy.env
include $(dpl)
export $(shell sed 's/=.*//' $(dpl))

# Image URL to use all building/pushing image targets
IMG ?= controller:latest
# ENVTEST_K8S_VERSION refers to the version of kubebuilder assets to be downloaded by envtest binary.
ENVTEST_K8S_VERSION = 1.29.0

COMMIT?=$(shell git rev-parse --short HEAD)
BUILD_TIME?=$(shell date -u '+%Y-%m-%d_%H:%M:%S')

PROJECT?=litmusportal-dex-server

# Get the currently used golang install path (in GOPATH/bin, unless GOBIN is set)
ifeq (,$(shell go env GOBIN))
GOBIN=$(shell go env GOPATH)/bin
else
GOBIN=$(shell go env GOBIN)
endif

# CONTAINER_TOOL defines the container tool to be used for building images.
# Be aware that the target commands are only tested with Docker which is
# scaffolded by default. However, you might want to replace it to use other
# tools. (i.e. podman)
CONTAINER_TOOL ?= docker

# Setting SHELL to bash allows bash commands to be executed by recipes.
# Options are set to exit when a recipe line exits non-zero or a piped command fails.
SHELL = /usr/bin/env bash -o pipefail
.SHELLFLAGS = -ec

.PHONY: all
all: build

##@ General

# The help target prints out all targets with their descriptions organized
# beneath their categories. The categories are represented by '##@' and the
# target descriptions by '##'. The awk command is responsible for reading the
# entire set of makefiles included in this invocation, looking for lines of the
# file as xyz: ## something, and then pretty-format the target and help. Then,
# if there's a line with ##@ something, that gets pretty-printed as a category.
# More info on the usage of ANSI control characters for terminal formatting:
# https://en.wikipedia.org/wiki/ANSI_escape_code#SGR_parameters
# More info on the awk command:
# http://linuxcommand.org/lc3_adv_awk.php

.PHONY: help
help: ## Display this help.
	@awk 'BEGIN {FS = ":.*##"; printf "\nUsage:\n  make \033[36m<target>\033[0m\n"} /^[a-zA-Z_0-9-]+:.*?##/ { printf "  \033[36m%-15s\033[0m %s\n", $$1, $$2 } /^##@/ { printf "\n\033[1m%s\033[0m\n", substr($$0, 5) } ' $(MAKEFILE_LIST)



# If you wish to build the manager image targeting other platforms you can use the --platform flag.
# (i.e. docker build --platform linux/arm64). However, you must enable docker buildKit for it.
# More info: https://docs.docker.com/develop/develop-images/build_enhancements/
.PHONY: docker-build
docker-build: ## Build docker image with the manager.
	$(CONTAINER_TOOL)  build -t $(APP_NAME) . --progress=plain --build-arg VERSION=${VERSION} --build-arg  COMMIT=${COMMIT} --build-arg BUILD_TIME=${BUILD_TIME} --build-arg PROJECT=${PROJECT} --build-arg BASE_IMAGE=${BASE_IMAGE} 


.PHONY: docker-build-nc
docker-build-nc: ## Build docker image with the manager.
	$(CONTAINER_TOOL)  build --no-cache -t $(APP_NAME) . --progress=plain --build-arg VERSION=${VERSION} --build-arg  COMMIT=${COMMIT} --build-arg BUILD_TIME=${BUILD_TIME} --build-arg PROJECT=${PROJECT} --build-arg BASE_IMAGE=${BASE_IMAGE} 


# Docker publish
.PHONY: docker-publish
docker-publish: docker-repo-login docker-publish-latest docker-publish-version ## Publish the `{version}` ans `latest` tagged containers to ECR

.PHONY: docker-publish-latest
docker-publish-latest: docker-tag-latest ## Publish the `latest` taged container to ECR
	@echo 'publish latest to $(DOCKER_REPO)'
	$(CONTAINER_TOOL)  push $(DOCKER_REPO)/$(APP_NAME):latest

.PHONY: docker-publish-version
docker-publish-version: docker-tag-version ## Publish the `{version}` taged container to ECR
	@echo 'publish $(VERSION) to $(DOCKER_REPO)'
	$(CONTAINER_TOOL)  push $(DOCKER_REPO)/$(APP_NAME):$(VERSION)

# Docker tagging
.PHONY: docker-tag
docker-tag: docker-tag-latest docker-tag-version ## Generate container tags for the `{version}` ans `latest` tags

.PHONY: docker-tag-latest
docker-tag-latest: ## Generate container `{version}` tag
	@echo 'create tag latest'
	$(CONTAINER_TOOL)  tag $(APP_NAME) $(DOCKER_REPO)/$(APP_NAME):latest

.PHONY: docker-tag-versaion
docker-tag-version: ## Generate container `latest` tag
	@echo 'create tag $(VERSION)'
	$(CONTAINER_TOOL)  tag $(APP_NAME) $(DOCKER_REPO)/$(APP_NAME):$(VERSION)


.PHONY: docker-inspect
docker-inspect: ## Generate container `latest` tag
	@echo 'inspect $(APP_NAME)'
	$(CONTAINER_TOOL)  history $(APP_NAME)
	$(CONTAINER_TOOL)  inspect $(APP_NAME)


.PHONY: docker-push
docker-push: ## Push docker image with the manager.
	$(CONTAINER_TOOL) push ${IMG}


.PHONY: docker-repo-login
docker-repo-login: ## Login to repo
	$(CONTAINER_TOOL) login	

.PHONY: docker-system-prune
docker-system-prune: ## Prune images
	$(CONTAINER_TOOL) system prune	

# PLATFORMS defines the target platforms for the manager image be built to provide support to multiple
# architectures. (i.e. make docker-buildx IMG=myregistry/mypoperator:0.0.1). To use this option you need to:
# - be able to use docker buildx. More info: https://docs.docker.com/build/buildx/
# - have enabled BuildKit. More info: https://docs.docker.com/develop/develop-images/build_enhancements/
# - be able to push the image to your registry (i.e. if you do not set a valid value via IMG=<myregistry/image:<tag>> then the export will fail)
# To adequately provide solutions that are compatible with multiple platforms, you should consider using this option.
PLATFORMS ?= linux/arm64,linux/amd64,linux/s390x,linux/ppc64le
.PHONY: docker-buildx
docker-buildx: ## Build and push docker image for the manager for cross-platform support
	# copy existing Dockerfile and insert --platform=${BUILDPLATFORM} into Dockerfile.cross, and preserve the original Dockerfile
	sed -e '1 s/\(^FROM\)/FROM --platform=\$$\{BUILDPLATFORM\}/; t' -e ' 1,// s//FROM --platform=\$$\{BUILDPLATFORM\}/' Dockerfile > Dockerfile.cross
	- $(CONTAINER_TOOL) buildx create --name project-v3-builder
	$(CONTAINER_TOOL) buildx use project-v3-builder
	- $(CONTAINER_TOOL) buildx build --push --platform=$(PLATFORMS) --tag ${IMG} -f Dockerfile.cross .
	- $(CONTAINER_TOOL) buildx rm project-v3-builder
	rm Dockerfile.cross


ifndef ignore-not-found
  ignore-not-found = false
endif



