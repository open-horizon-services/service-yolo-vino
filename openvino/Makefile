# A plugin using openvino (requires Movidius VPU hardware, software, and config)

# Include the make file containing all the check-* targets
include ../../checks.mk

# Give this service a name, version number, and pattern name
SERVICE_NAME:="openvino"
SERVICE_VERSION:="1.1.0"
PATTERN_NAME:="pattern-openvino"

# These statements automatically configure some environment variables
ARCH:=$(shell ../../helper -a)

# Leave blank for open DockerHub containers
# CONTAINER_CREDS:=-r "registry.wherever.com:myid:mypw"
CONTAINER_CREDS:=

build: check-dockerhubid
	@echo "Building the OpenVino plugin for Movidius hardware"
	docker build -t $(DOCKERHUB_ID)/$(SERVICE_NAME)_$(ARCH):$(SERVICE_VERSION) -f ./Dockerfile.$(ARCH) .; \

run: check-dockerhubid
	-docker network create mqtt-net 2>/dev/null || :
	-docker network create cam-net 2>/dev/null || :
	-docker rm -f $(SERVICE_NAME) 2>/dev/null || :
	docker run --rm -d \
          --privileged -v /dev:/dev \
          --name $(SERVICE_NAME) \
          -e OPENVINO_PLUGIN=MYRIAD \
          --network=host \
          $(DOCKERHUB_ID)/$(SERVICE_NAME)_$(ARCH):$(SERVICE_VERSION)

dev: check-dockerhubid
	-docker network create mqtt-net 2>/dev/null || :
	-docker network create cam-net 2>/dev/null || :
	-docker rm -f $(SERVICE_NAME) 2>/dev/null || :
	docker run -it -v `pwd`:/outside \
          --privileged -v /dev:/dev \
          --name $(SERVICE_NAME) \
          -e OPENVINO_PLUGIN=MYRIAD \
          --privileged -v /dev:/dev \
          --network=host \
          $(DOCKERHUB_ID)/$(SERVICE_NAME)_$(ARCH):$(SERVICE_VERSION) /bin/bash

stop: check-dockerhubid
	-docker rm -f ${SERVICE_NAME} 2>/dev/null || :

test:
	@echo "Attempting to test inferencing on an image from the restcam service..."
	curl -sS http://127.0.0.1:80/detect?url='http%3A%2F%2Flocalhost%3A8888%2F' | jq --raw-output --join-output '.detect.image' | base64 -d > ./inferred.jpg
	-ls -l ./inferred.jpg

clean: check-dockerhubid
	-docker rm -f ${SERVICE_NAME} 2>/dev/null || :
	-docker rmi $(DOCKERHUB_ID)/$(SERVICE_NAME)_$(ARCH):$(SERVICE_VERSION) 2>/dev/null || :

publish-service:
	@ARCH=$(ARCH) \
	    SERVICE_NAME="$(SERVICE_NAME)" \
	    SERVICE_VERSION="$(SERVICE_VERSION)"\
	    SERVICE_CONTAINER="$(DOCKERHUB_ID)/$(SERVICE_NAME)_$(ARCH):$(SERVICE_VERSION)" \
	    hzn exchange service publish -O $(CONTAINER_CREDS) -P -f service.json --public=true

publish-pattern:
	@ARCH=$(ARCH) \
	    SERVICE_NAME="$(SERVICE_NAME)" \
	    SERVICE_VERSION="$(SERVICE_VERSION)"\
	    PATTERN_NAME="$(PATTERN_NAME)" \
	    hzn exchange pattern publish -f pattern.json

agent-run:
	hzn register --pattern "${HZN_ORG_ID}/$(PATTERN_NAME)"

agent-stop:
	hzn unregister -f

.PHONY: build run dev stop test clean publish-service publish-pattern agent-run agent-stop

