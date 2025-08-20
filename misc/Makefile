.PHONY: build release clean gen

# Get the current HEAD commit
HEAD_COMMIT := $(shell git rev-parse HEAD)

# Set HEAD_COMMIT_SHORT to short commit id
HEAD_COMMIT_SHORT := $(shell git rev-parse --short HEAD)

# Check if the HEAD commit has a tag
HEAD_TAG := $(shell git tag --points-at $(HEAD_COMMIT))

# set HEAD_DATE to the date of the HEAD commit
HEAD_DATE := $(shell git show -s --date=format:'%Y%m%d%H%M%S' --format=%cd $(HEAD_COMMIT))

# set BUILD_VERSION to the tag name if it exists, otherwise "0.0.0" is used for canary build
ifeq ($(HEAD_TAG),)
BUILD_VERSION := "0.0.0.$(HEAD_COMMIT_SHORT)-$(HEAD_DATE)"
else
BUILD_VERSION := "$(HEAD_TAG)"
endif

# default build platform and architecture
BUILD_PLATFORM := "windows"
BUILD_ARCH := "amd64"

# set BUILD_DATE to the current date
BUILD_DATE := $(shell date "+%Y-%m-%dT%H:%M:%S%z")

BUILD_DATE_SANITIZED := $(shell date -u "+%Y-%m-%dT%H:%M:%SZ")

# edition name, will be provided by the CI/CD system,
# here we use "canary" if build is triggered manually
EDITION ?= "canary"

# set binary variables override
GO_BUILD_LDFLAGS := "-X 'main.Version=$(VERSION_NAME)' -X 'main.BuildDate=$(BUILD_DATE)'"

# protobuf source directory
PROTO_DIR=proto
# grpc proto out directory
OUT_DIR=app/proto

# alias for all proto files
ALL_PROTO_FILES=$(shell find $(PROTO_DIR) -type f -name "*.proto")
PROTO_FILES=$(wildcard proto/*.proto)

all: clean gen build

# push proto to separated repository
push:
	@cd proto/ && git push

# clear build directory and generated protobuf and grpc related files
clean:
	rm -rf ./build/*
	find . -name "*.gen.go" -type f -delete
	find . -name "*.pb.go" -type f -delete
	find . -name "*.pb.gw.go" -type f -delete

gen:
	@echo "Need protoc, go-gen-grpc, go-gen-grpc-gateway installed"
	protoc --go_out=$(OUT_DIR) --go_opt=paths=source_relative --go-grpc_out=$(OUT_DIR) --go-grpc_opt=paths=source_relative --grpc-gateway_out=$(OUT_DIR) --grpc-gateway_opt=paths=source_relative --grpc-gateway_opt=generate_unbound_methods=true -I $(PROTO_DIR) $(PROTO_FILES)
#	@echo "Generate gorm models, need gorm.io/gentool installed at ./tools/gentool"
#	./tools/gentool -dsn "root:root@tcp(127.0.0.1:3306)/mydb1?charset=utf8mb4&parseTime=True&loc=Local" -onlyModel -fieldWithIndexTag -fieldWithTypeTag -fieldSignable -outPath "app/models/dao" -modelPkgName "dao"
# personal flavor for history table to end in _YYYY_MM suffix
#	@echo "Delete history table"
#	find ./app/models/dao -type f -regex ".*[0-9].*" -delete
	GOOS=$(BUILD_PLATFORM) GOARCH=$(BUILD_ARCH) go generate ./...

dep:
	@echo "Need Graphviz, and github.com/kisielk/godepgraph installed at ./tools/godepgraph"
	./tools/godepgraph -s -p github.com,golang.org,gopkg.in,go.uber.org,gorm.io,go.opencensus.io,google.golang.org . | dot -Tpng -o dep.png

build: build-config
	go mod tidy
	GOOS=$(BUILD_PLATFORM) GOARCH=$(BUILD_ARCH) go build -trimpath -ldflags "-X 'main.Version=$(BUILD_VERSION)-$(EDITION)' -X 'main.BuildDate=$(BUILD_DATE)'" -o ./build/main .

build-config:
	cp -r ./config/ ./build/
	cp *.crt ./build/
	cp *.key ./build/
	mkdir -p "./build/logs"

release: build-config
	$(eval EDITION=release)
	go mod tidy
	GOOS=$(BUILD_PLATFORM) GOARCH=$(BUILD_ARCH) go build -trimpath -ldflags "-X 'main.Version=$(BUILD_VERSION)-$(EDITION)' -X 'main.BuildDate=$(BUILD_DATE)'" -o ./build/main .

# set alias
clear: clean

regen: clean gen
