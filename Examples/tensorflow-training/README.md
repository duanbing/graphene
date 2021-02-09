### Prerequirements

Install [miniconda](https://docs.conda.io/en/latest/miniconda.html) and activate python 3.6 by 
```
conda create --name py36 python=3.6
conda activate py36
```

Test:
```
pip install --user numpy==1.18.5 tensorflow==v1.15.2 wrapt keras_applications keras_preprocessing

make distclean && make SGX=1 distclean
make SGX=1 run-graphene

```

### Run

```
make SGX=1 DEBUG=1 run-graphene
```
