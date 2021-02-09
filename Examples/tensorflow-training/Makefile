manifests = $(addsuffix .manifest,label_image)
exec_target = $(manifests)

GRAPHENEDIR ?= ../..
SGX_SIGNER_KEY ?= $(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/enclave-key.pem
TF_DIR ?= tensorflow

PYTHON3 ?= /usr/bin/python3

ifeq ($(DEBUG),1)
GRAPHENE_LOG_LEVEL = debug
else
GRAPHENE_LOG_LEVEL = error
endif

#GIT_COMMIT ?= v1.9.0
#TAR_SHA256 ?= ffc3151b06823d57b4a408261ba8efe53601563dfe93af0866751d4f6ca5068c

.PHONY: all
all: label_image.manifest
ifeq ($(SGX),1)
all: label_image.manifest.sgx label_image.sig label_image.token
endif

include ../../Scripts/Makefile.configs

INCEPTION_HASH=b1a1f91276e48a9ddf0cb0d854f044ebfbf985dc2c2cedceb52b3d668894299a
inception_v3.tflite:
	$(GRAPHENEDIR)/Scripts/download --output inception_v3_2018_04_27.tgz --sha256 $(INCEPTION_HASH)\
	       --url https://storage.googleapis.com/download.tensorflow.org/models/tflite/model_zoo/upload_20180427/inception_v3_2018_04_27.tgz
	tar xfz inception_v3_2018_04_27.tgz

$(GRAPHENEDIR)/Runtime/libgcc_s.so.1:
	cp $(ARCH_LIBDIR)/libgcc_s.so.1 $@

$(GRAPHENEDIR)/Runtime/libstdc++.so.6:
	cp /usr/$(ARCH_LIBDIR)/libstdc++.so.6 $@

#label_image.manifest: libtensorflow_framework.so label_image inception_v3.tflite labels.txt image.bmp
label_image.manifest: $(GRAPHENEDIR)/Runtime/libgcc_s.so.1 $(GRAPHENEDIR)/Runtime/libstdc++.so.6
label_image.manifest: label_image.manifest.template
	sed -e 's|$$(GRAPHENEDIR)|'"$(GRAPHENEDIR)"'|g' \
		-e 's|$$(GRAPHENE_LOG_LEVEL)|'"$(GRAPHENE_LOG_LEVEL)"'|g' \
		-e 's|$$(PYTHON3)|'"$(PYTHON3)"'|g' \
		-e 's|$$(ARCH_LIBDIR)|'"$(ARCH_LIBDIR)"'|g' \
		-e 's|$$(HOME)|'"$(HOME)"'|g' \
		$< > $@

label_image.manifest.sgx: label_image.manifest
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-sign \
		-libpal $(GRAPHENEDIR)/Runtime/libpal-Linux-SGX.so \
		-key $(SGX_SIGNER_KEY) \
		-manifest $< -output $@
	$(GRAPHENEDIR)/Pal/src/host/Linux-SGX/signer/pal-sgx-get-token \
		-output label_image.token -sig label_image.sig

.PHONY: check
check: all
	$(GRAPHENEDIR)/Runtime/pal_loader label_image model/lr.py 

.PHONY: run-native
run-native: all
	python3 model/lr.py

.PHONY: run-graphene
run-graphene: all
	$(GRAPHENEDIR)/Runtime/pal_loader label_image model/lr.py
	#GDB=1 $(GRAPHENEDIR)/Runtime/pal_loader label_image 

.PHONY: clean
clean:
	$(RM) label_image.manifest label_image.manifest.sgx label_image.sig label_image.token

.PHONY: distclean
distclean: clean
	#$(RM) -r label_image
	$(RM) inception_v3_2018_04_27.tgz inception_v3.pb inception_v3.tflite labels.txt image.bmp
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
