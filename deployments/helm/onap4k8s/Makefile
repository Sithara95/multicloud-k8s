# Copyright © 2017 Amdocs, Bell Canada
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#       http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

PARENT_CHART := onap4k8s
COMMON_CHARTS_DIR := charts/common
ROOT_DIR := $(shell dirname $(realpath $(lastword $(MAKEFILE_LIST))))
OUTPUT_DIR := $(ROOT_DIR)/dist
PACKAGE_DIR := $(OUTPUT_DIR)/packages
SECRET_DIR := $(OUTPUT_DIR)/secrets

EXCLUDES := config oneclick readiness test dist helm $(PARENT_CHART) dcae
HELM_CHARTS := $(filter-out $(EXCLUDES), $(patsubst %/.,%,$(wildcard */.))) $(PARENT_CHART)

.PHONY: $(EXCLUDES) $(HELM_CHARTS)

all: $(COMMON_CHARTS_DIR) $(HELM_CHARTS) plugins

$(COMMON_CHARTS):
	@echo "\n[$@]"
	@make package-$@

$(HELM_CHARTS):
	@echo "\n[$@]"
	@make package-$@

make-%:
	@if [ -f $*/Makefile ]; then make -C $*; fi

dep-%: make-%
	@if [ -f $*/requirements.yaml ]; then helm dep up $*; fi

lint-%: dep-%
	@if [ -f $*/Chart.yaml ]; then helm lint $*; fi

package-%: lint-%
	@mkdir -p $(PACKAGE_DIR)
	@if [ -f $*/Chart.yaml ]; then helm package -d $(PACKAGE_DIR) $*; fi
	@helm repo index $(PACKAGE_DIR)

clean:
	@rm -f */requirements.lock
	@find . -type f -name '*.tgz' -delete
	@rm -rf $(PACKAGE_DIR)/*
	@rm -rf $(OUTPUT_DIR)

# start up a local helm repo to serve up helm chart packages
repo:
	@mkdir -p $(PACKAGE_DIR)
	@helm serve --repo-path $(PACKAGE_DIR) &
	@helm repo index $(PACKAGE_DIR)
	@helm repo add onap4k8s http://127.0.0.1:8879

# stop local helm repo
repo-stop:
	@pkill helm
	@helm repo remove onap4k8s
%:
	@:
