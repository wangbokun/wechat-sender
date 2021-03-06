PKGS ?= $(shell glide nv)
PKG_FILES ?= *.go

# The linting tools evolve with each Go version, so run them only on the latest
# stable release.
GO_VERSION := $(shell go version | cut -d " " -f 3)
GO_MINOR_VERSION := $(word 2,$(subst ., ,$(GO_VERSION)))
LINTABLE_MINOR_VERSIONS := 12
ifneq ($(filter $(LINTABLE_MINOR_VERSIONS),$(GO_MINOR_VERSION)),)
SHOULD_LINT := true
endif

.PHONY: dependencies
dependencies:
	@echo "Installing Glide and locked dependencies..."
	glide --version || go get -u -f github.com/Masterminds/glide
	glide install
ifdef SHOULD_LINT
	@echo "Installing golint..."
	go get -u golang.org/x/lint/golint
else
	@echo "Not installing golint, since we don't expect to lint on" $(GO_VERSION)
endif

.PHONY: test
test:
	@.build/test.sh

# Disable printf-like invocation checking due to testify.assert.Error()
VET_RULES := -printf=false

.PHONY: lint
lint:
ifdef SHOULD_LINT
	@rm -rf lint.log
	@echo "Checking formatting..."
	@gofmt -d -s $(PKG_FILES) 2>&1 | tee lint.log
	@echo "Installing test dependencies..."
	@go test -i $(PKGS)
	@echo "Checking vet..."
	@go vet ./... 2>&1 | tee -a lint.log
	@echo "Checking lint..."
	@$(foreach dir,$(PKGS),golint $(dir) 2>&1 | tee -a lint.log;)
	@echo "Checking for unresolved FIXMEs..."
	@git grep -i fixme | grep -v -e vendor -e Makefile -e .md | tee -a lint.log
	@echo "Checking for license headers..."
	@DRY_RUN=1 .build/check_license.sh | tee -a lint.log
	@[ ! -s lint.log ]
else
	@echo "Skipping linters on" $(GO_VERSION)
endif

.PHONY: ci
ci: SHELL := /bin/bash
ci: test lint
	bash <(curl -s https://codecov.io/bash)

