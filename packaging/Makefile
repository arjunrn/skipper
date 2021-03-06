.PHONY: docker-push

VERSION            ?= $(shell git rev-parse HEAD)
REGISTRY           ?= registry-write.opensource.zalan.do/pathfinder
IMAGE              ?= $(REGISTRY)/skipper:$(VERSION)
LIGHTSTEP_IMAGE    ?= $(REGISTRY)/skipper-lightstep:$(VERSION)
ALPINE_BUILD_IMAGE ?= $(REGISTRY)/skipper-alpine-build:latest
PACKAGE            ?= github.com/zalando/skipper
CGO_ENABLED        ?= 0

default: docker-build

skipper:
	GOOS=linux GOARCH=amd64 CGO_ENABLED=$(CGO_ENABLED) go build $(PACKAGE)/cmd/skipper

eskip:
	GOOS=linux GOARCH=amd64 CGO_ENABLED=$(CGO_ENABLED) go build $(PACKAGE)/cmd/eskip

clean:
	rm -rf skipper eskip build/

docker-build: clean skipper eskip
	docker build -t $(IMAGE) .

docker-push:
	docker push $(IMAGE)

plugins:
	CUR=$(shell pwd) ; GOPATH=$(shell pwd)/build ;\
		go get -d github.com/skipper-plugins/opentracing ;\
		cd build/src/github.com/skipper-plugins/opentracing ;\
		glide install -v;\
		GOOS=linux GOARCH=amd64 CGO_ENABLED=1 make ;\
		cp build/*.so $$CUR/build/

alpine.build: docker-alpine-build
	docker run --rm -v $(shell pwd)/..:/go/src/$(PACKAGE) \
		-w /go/src/$(PACKAGE)/packaging $(ALPINE_BUILD_IMAGE) \
		make skipper eskip

alpine.plugins: docker-alpine-build
	docker run --rm -v $(shell pwd)/..:/go/src/$(PACKAGE) \
		-w /go/src/$(PACKAGE)/packaging $(ALPINE_BUILD_IMAGE) \
		make plugins

docker-alpine-build:
	docker build --rm -t $(ALPINE_BUILD_IMAGE) -f Dockerfile.alpine .

docker-lightstep-build: alpine.build alpine.plugins
	docker build -t $(LIGHTSTEP_IMAGE) -f Dockerfile.lightstep .

docker-lightstep-push:
	docker push $(LIGHTSTEP_IMAGE)
