### Prerequirements

Install [miniconda](https://docs.conda.io/en/latest/miniconda.html) and activate python 3.6 by 
```
conda create --name py36 python=3.6
conda activate py36
```

### Run

#### Sample
```
pip install --user numpy==1.18.5 tensorflow==v1.15.2 wrapt keras_applications keras_preprocessing

make distclean && make SGX=1 distclean
make SGX=1 run-graphene

```

#### Distributed Training

Train a LR model in 2 PS' and 2 workers network. Execute each command below in a single termial.
```
make clean && make SGX=1 distclean && make SGX=1 run-graphene-ps0
make clean && make SGX=1 distclean && make SGX=1 run-graphene-ps1
make clean && make SGX=1 distclean && make SGX=1 run-graphene-w0
make clean && make SGX=1 distclean && make SGX=1 run-graphene-w1
```
