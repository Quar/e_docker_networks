SHELL := /bin/bash

remote ?= qtprobe

image-name ?= nettestimg
container-name ?= nettest
container-entrypoint ?= /bin/sh

docker-run-args ?= --cap-add=NET_ADMIN --net=test

basename := $(shell basename $(PWD))
dirname := $(shell dirname $(PWD))

remote-dir := ~/_test_
DOCKER_DIR ?= .
COMPOSE_DIR ?= .

host-path = $(PWD)
remote-path = $(remote-dir)/$(basename)
ssh-path = $(remote):"$(remote-path)"

rsync-cmd := rsync -azt

exclude-files := \
  '*.swp' \
  '.*' \

exclude-cmd := $(addprefix --exclude ,$(exclude-files))


rsync = $(rsync-cmd) $(exclude-cmd)


.PHONY: echo
echo:
	@echo "==> exclude:"
	@echo "$(exclude)"



.PHONY: ssh-mkdir-p
ssh-mkdir-p:
	@ssh $(remote) '[[ -d $(remote-path) ]] || mkdir -p $(remote-path)'


.PHONY: push
push: ssh-mkdir-p
	@echo '==> Push files to remote $(ssh-path) ...'
	@$(rsync) $(host-path)/ $(ssh-path)


.PHONY: push-force
push-force:
	@echo '==> Force push files to remote $(ssh-path) ...'
	@$(rsync) --force $(host-path)/ $(ssh-path)


.PHONY: pull
pull:
	@echo '==> Pull files from remote $(ssh-path) ...'
	@$(rsync) $(ssh-path)/ $(host-path)


.PHONY: sync
sync: pull push


.PHONY: docker-build
docker-build:
	@echo '==> Check docker image $(image-name) ...'
	@docker build -t $(image-name) $(DOCKER_DIR)

.PHONY: docker-up
docker-up: docker-build
	@echo '==> Running container $(container-name) detached ...'
	@[[ $$(docker ps -f "name=$(container-name)" --format '{{.Names}}') \
		== \
		$(container-name) \
		]] || \
	 docker run -itd --name=$(container-name) $(docker-run-args) $(image-name)


.PHONY: docker-exec
docker-exec:
	@echo '==> Allocate tty ...'
	@docker exec -it $(container-name) $(container-entrypoint)


.PHONY: docker-run
docker-run: docker-up docker-exec


.PHONY: docker-rerun
docker-rerun: docker-stoprm docker-build docker-run


.PHONY: docker-ssh-exec
docker-ssh-exec: push
	@echo "==> docker exec on $(remote):$(remote-path)"
	@echo "    Expecting Dockerfile in path $(DOCKER_DIR) ..."
	@ssh -t $(remote) 'bash -l -c "docker exec -it $(container-name) $(container-entrypoint)"'


.PHONY: docker-ssh-run
docker-ssh-run: push
	@echo "==> Running docker on $(remote):$(remote-path)"
	@echo "    Expecting Dockerfile in path $(DOCKER_DIR) ..."
	@ssh -t $(remote) 'export DOCKER_DIR=$(DOCKER_DIR) && cd $(remote-path) && bash -l -c "make docker-run"'


.PHONY: docker-stoprm
docker-stoprm:
	@echo "==> Stop and remove container $(container-name) ..."
	@docker stop $(container-name) || true \
	  && \
	  docker rm $(container-name) || true


.PHONY: docker-ssh-rerun
docker-ssh-rerun: push
	@echo "==> Re-running docker on $(remote):$(remote-path)"
	@echo "    Expecting Dockerfile in path $(DOCKER_DIR) ..."
	@ssh -t $(remote) 'export DOCKER_DIR=$(DOCKER_DIR) && cd $(remote-path) && bash -l -c "make docker-rerun"'


.PHONY: docker-compose-up
docker-compose-up:
	@echo "==> Running \`docker-compose up -d\` ..."
	@docker-compose up -d


.PHONY: docker-ssh-compose-up
docker-ssh-compose-up: push
	@echo "==> Running on $(remote) with \`docker-compose up -d\` ..."
	@ssh -t $(remote) "export COMPOSE_DIR=$(COMPOSE_DIR) && cd $(remote-path) && bash -l -c 'make docker-compose-up'"


.PHONY: docker-compose-down
docker-compose-down:
	@echo "==> Running \`docker-compose down\` ..."
	@docker-compose down


.PHONY: docker-ssh-compose-down
docker-ssh-compose-down: push
	@echo "==> Running on $(remote) with \`docker-compose down\` ..."
	@ssh -t $(remote) "export COMPOSE_DIR=$(COMPOSE_DIR) && cd $(remote-path) && bash -l -c 'make docker-compose-down'"


.PHONY: docker-compose-update
docker-compose-update: docker-compose-down docker-compose-up


.PHONY: docker-ssh-compose-update
docker-ssh-compose-update: docker-ssh-compose-down docker-ssh-compose-up


