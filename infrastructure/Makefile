SHELL := /bin/bash

AWS_ACCESS_KEY_ID=$(shell cat build-files/env/AWS_ACCESS_KEY_ID)
AWS_SECRET_ACCESS_KEY=$(shell cat build-files/env/AWS_SECRET_ACCESS_KEY)
AWS_DEFAULT_REGION=$(shell cat build-files/env/AWS_DEFAULT_REGION)

#####################################################################
#                          Docker Targets                           #
#####################################################################

DOCKER_IMAGE_NAME=solar-data-logger-api-build
DOCKER_CONTAINER=$(shell cat $(DOCKER_IMAGE_NAME).cid 2> /dev/null)
DOCKERFILE=build-files/docker/Dockerfile

docker-exec=docker exec \
	--interactive \
	--tty \
	-e AWS_ACCESS_KEY_ID=$(AWS_ACCESS_KEY_ID) \
	-e AWS_SECRET_ACCESS_KEY=$(AWS_SECRET_ACCESS_KEY) \
	-e AWS_DEFAULT_REGION=$(AWS_DEFAULT_REGION) \
	$(DOCKER_CONTAINER) \
	/bin/bash

.PHONY += docker-build
docker-build:
	docker build \
		--tag $(DOCKER_IMAGE_NAME) \
		--file $(DOCKERFILE) \
		.

.PHONY += docker-start
docker-start: docker-build
	@ if test ! -f "$(DOCKER_IMAGE_NAME).cid"; then \
		docker run \
			--cidfile $(DOCKER_IMAGE_NAME).cid \
			--interactive \
			--tty \
			--detach \
			--volume $(shell pwd):/data \
			$(DOCKER_IMAGE_NAME) \
			--rm \
			/bin/bash; \
	fi

.PHONY += docker-attach
docker-attach: docker-start
	$(docker-exec)

.PHONY += docker-kill
docker-kill:
	@ if test -f "$(DOCKER_IMAGE_NAME).cid"; then \
		docker kill $(DOCKER_CONTAINER) || true; \
	fi
	@ rm $(DOCKER_IMAGE_NAME).cid 2> /dev/null || true;

.PHONY += docker-clean
docker-clean: docker-kill
	docker rmi --force $(DOCKER_IMAGE_NAME) || true;

#####################################################################
#                    Virtual Environment Targets                    #
#####################################################################

VENV_PATH=.venv
VENV_REQUIREMENTS=build-files/virtual-environment/requirements.txt

.PHONY+=create-virtual-environment
create-virtual-environment: docker-start
	@ $(docker-exec) -c " \
		virtualenv $(VENV_PATH) \
	"

#####################################################################
#                          Terraform Targets                        #
#####################################################################

.PHONY+=terraform-init
terraform-init: docker-start
	@ $(docker-exec) -c " \
		terraform init \
	"

.PHONY+=terraform-plan
terraform-plan: docker-start terraform-init
	@ $(docker-exec) -c " \
		terraform plan \
	"

.PHONY+=terraform-apply
terraform-apply: docker-start terraform-init
	@ $(docker-exec) -c " \
		terraform apply \
	"

.PHONY+=terraform-destroy
terraform-destroy: docker-start terraform-init
	@ $(docker-exec) -c " \
		terraform destroy \
	"