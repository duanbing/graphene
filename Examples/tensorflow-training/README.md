### Prerequirements

Ubuntu18.04, Docker and GrapheneSGX

#### Before we go
We build this sample in docker container, and we assume you had installed the graphene sgx driver.

Then get jailed into docker container:
```
cd /path/to/same/directory/as/graphene
docker run -it --cap-add=SYS_PTRACE --device /dev/gsgx --device /dev/sgx/enclave --security-opt seccomp=unconfined --name=tf --net=host -v $PWD:/app -w /app  ubuntu:18.04 bash

```

and install all the dependencies below:

```
apt-get install -y build-essential  wget gawk bison 
apt-get install -y python3-protobuf
apt-get install -y libprotobuf-c-dev protobuf-c-compiler \
   libcurl4-openssl-dev
apt install -y libnss-mdns libnss-myhostname

```

then install tensorflow and it's dependencies as below.

```
python3 -m pip install --upgrade pip setuptools
pip install --user numpy==1.18.5 tensorflow==v1.15.2 wrapt keras_applications keras_preprocessing
```
Now, everything you need to run tensorflow training is ready.

Ok, what you had done looks a bit complicated, you also can try our released docker image like this: 

```
cd /path/to/same/directory/as/graphene
docker run -it --cap-add=SYS_PTRACE --device /dev/gsgx --device /dev/sgx/enclave --security-opt seccomp=unconfined --name=tf --net=host -v $PWD:/app -w /app  duanbing0613/ubuntu1804:sgx-tf2 bash
```
#### Compile Graphene

```
cd graphene
make distclean && make SGX=1 distclean
ISGX_DRIVER_PATH= make SGX=1 DEBUG=1

# optional
openssl genrsa -3 -out enclave-key.pem 3072

export SGX_SIGNER_KEY=$PWD/enclave-key.pem
```

### Run TF Training

#### Eager Mode 
```
make distclean && make SGX=1 distclean
make SGX=1 run-graphene
```

#### Distributed Training Mode

Train a LR model in 2 PS' and 2 workers network. Execute each command below in a single termial.
```
make clean && make SGX=1 distclean && make SGX=1 run-graphene-ps0
make clean && make SGX=1 distclean && make SGX=1 run-graphene-ps1
make clean && make SGX=1 distclean && make SGX=1 run-graphene-w0
make clean && make SGX=1 distclean && make SGX=1 run-graphene-w1
```
