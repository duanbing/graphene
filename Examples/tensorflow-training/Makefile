manifests = $(addsuffix .manifest,tf)
exec_target = $(manifests)

GRAPHENEDIR ?= ../..
SGX_SIGNER_KEY ?= $(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/enclave-key.pem

PYTHON3 ?= /usr/bin/python3

ifeq ($(DEBUG),1)
GRAPHENE_LOG_LEVEL = debug
else
GRAPHENE_LOG_LEVEL = error
endif

#GIT_COMMIT ?= v1.9.0
#TAR_SHA256 ?= ffc3151b06823d57b4a408261ba8efe53601563dfe93af0866751d4f6ca5068c

.PHONY: all
all: tf.manifest
ifeq ($(SGX),1)
all: tf.manifest.sgx tf.sig tf.token
endif

include ../../Scripts/Makefile.configs

$(GRAPHENEDIR)/Runtime/libgcc_s.so.1:
	cp $(ARCH_LIBDIR)/libgcc_s.so.1 $@

$(GRAPHENEDIR)/Runtime/libstdc++.so.6:
	cp /usr/$(ARCH_LIBDIR)/libstdc++.so.6 $@

#tf.manifest: libtensorflow_framework.so tf inception_v3.tflite labels.txt tf.bmp
tf.manifest: $(GRAPHENEDIR)/Runtime/libgcc_s.so.1 $(GRAPHENEDIR)/Runtime/libstdc++.so.6
tf.manifest: tf.manifest.template
	sed -e 's|$$(GRAPHENEDIR)|'"$(GRAPHENEDIR)"'|g' \
		-e 's|$$(GRAPHENE_LOG_LEVEL)|'"$(GRAPHENE_LOG_LEVEL)"'|g' \
		-e 's|$$(PYTHON3)|'"$(PYTHON3)"'|g' \
		-e 's|$$(ARCH_LIBDIR)|'"$(ARCH_LIBDIR)"'|g' \
		-e 's|$$(HOME)|'"$(HOME)"'|g' \
		$< > $@

tf.manifest.sgx: tf.manifest
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-sign \
		-libpal $(GRAPHENEDIR)/Runtime/libpal-Linux-SGX.so \
		-key $(SGX_SIGNER_KEY) \
		-manifest $< -output $@
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-get-token \
		-output tf.token -sig tf.sig

.PHONY: check
check: all
	$(GRAPHENEDIR)/Runtime/pal_loader tf  model/end-2-end/train.py --task_index=0 --job_name=ps 

.PHONY: run-native
run-native: all
	python3 model/lr.py

.PHONY: run-graphene
run-graphene: all
	$(GRAPHENEDIR)/Runtime/pal_loader tf model/lr.py

.PHONY: run-graphene-ps0
run-graphene-ps0: all
	$(GRAPHENEDIR)/Runtime/pal_loader tf model/end-2-end/train.py --task_index=0 --job_name=ps 

.PHONY: run-graphene-ps1
run-graphene-ps1: all
	$(GRAPHENEDIR)/Runtime/pal_loader tf model/end-2-end/train.py --task_index=1 --job_name=ps 

.PHONY: run-graphene-w0
run-graphene-w0: all
	$(GRAPHENEDIR)/Runtime/pal_loader tf model/end-2-end/train.py --task_index=0 --job_name=worker

.PHONY: run-graphene-w1
run-graphene-w1: all
	$(GRAPHENEDIR)/Runtime/pal_loader tf model/end-2-end/train.py --task_index=1 --job_name=worker

.PHONY: clean
clean:
	$(RM) tf.manifest tf.manifest.sgx tf.sig tf.token

.PHONY: distclean
distclean: clean
	#$(RM) -r tf
	$(RM) -r tensorflow.tar.gz libtensorflow_framework.so bazel-*-installer-linux-*.sh

BAZEL_INSTALLER_HASH=17ab70344645359fd4178002f367885e9019ae7507c9c1ade8220f3628383444
.PHONY: install-dependencies-ubuntu
install-dependencies-ubuntu:
	apt-get update
	apt-get install -y python-dev python-pip wget git numpy mock
	# https://docs.bazel.build/versions/master/install-ubuntu.html
	apt-get install -y pkg-config zip g++ zlib1g-dev unzip python
	$(GRAPHENEDIR)/Scripts/download --output bazel-0.16.1-installer-linux-x86_64.sh --sha256 $(BAZEL_INSTALLER_HASH)\
		--url https://github.com/bazelbuild/bazel/releases/download/0.16.1/bazel-0.16.1-installer-linux-x86_64.sh
	chmod +x bazel-0.16.1-installer-linux-x86_64.sh
	./bazel-0.16.1-installer-linux-x86_64.sh --user

.PHONY: install-dependencies-fedora
install-dependencies-fedora:
	dnf -y install python3-devel python3-pip wget git pkg-config zip gcc-g++ zlib-devel unzip
	$(GRAPHENEDIR)/Scripts/download --output bazel-0.16.1-installer-linux-x86_64.sh --sha256 $(BAZEL_INSTALLER_HASH)\
		--url https://github.com/bazelbuild/bazel/releases/download/0.16.1/bazel-0.16.1-installer-linux-x86_64.sh
	chmod +x bazel-0.16.1-installer-linux-x86_64.sh
	./bazel-0.16.1-installer-linux-x86_64.sh --user
