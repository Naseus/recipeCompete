SHELL:=/bin/bash

ifdef CI
TRUNK=./trunk
else
TRUNK=trunk
endif

TAILWIND_FLAGS=-c ./tailwind.config.js -o ./tailwind.css
TAILWIND_DEV_FLAGS=--watch
TAILWIND_PROD_FLAGS=--minify

# frontend static site container (nginx-based)
DOCKER_ACCOUNT=jdevries3133
CONTAINER_NAME=recipe_front
TAG?=$(shell git describe --tags)
CONTAINER=$(DOCKER_ACCOUNT)/$(CONTAINER_NAME):$(TAG)



.PHONY: setup
setup:
	@# check installation of npm package `tailwindcss`
	@which tailwindcss > /dev/null; \
	if [[ $$? -ne 0 ]]; then \
		echo "fatal: must install tailwindcss with ``npm install --global tailwindcss``"; \
		exit 1; \
	fi
ifdef CI
	# download a trunk binary to the current working directory
	wget -qO- https://github.com/thedodd/trunk/releases/download/v0.15.0/trunk-x86_64-unknown-linux-gnu.tar.gz | tar -xzf-
	rustup target add wasm32-unknown-unknown
endif


.PHONY: develop
develop: setup
	@# TODO: this rule is buggy because it seems like trunk is continuously
	@# rebuilding during the entire time that tailwind is generating the CSS.
	@# like, it it takes 100ms for tailwind to generate css and 2ms for trunk
	@# to rebuild, trunk will rebuild over and over again for that entire
	@# 100ms while the tailwind.css file is changing
	concurrently -n css,trunk \
		"tailwindcss $(TAILWIND_FLAGS) $(TAILWIND_DEV_FLAGS)" \
		"$(TRUNK) serve"


.PHONY: clean
clean:
	$(TRUNK) clean
	cargo clean
	rm -rf tailwind.css


.PHONY: build
build: setup clean
	tailwindcss $(TAILWIND_FLAGS) $(TAILWIND_PROD_FLAGS)
	$(TRUNK) build --release
	docker buildx build --platform linux/amd64 --push -t $(CONTAINER) .


.PHONY: deploy
deploy: build
ifdef CI
	terraform init -input=false
endif
	terraform apply -auto-approve


.PHONY: test
test:
	IS_PRODUCTION=true cargo test
	cargo test
