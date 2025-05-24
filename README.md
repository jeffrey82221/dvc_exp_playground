# dvc_exp_playground
Study the experimenting feature of DVC (This is a subproject of dvc_playground)

# Reference: 
https://dvc.org/doc/start/experiments/experiment-pipelines

# Tutorial 

This tutorial is based on the example repo provided by DVC: 

https://github.com/iterative/example-get-started-experiments/tree/main

# Steps:

## Step1: setup a runnable pipeline 

1. setup virtual environment 

```
python3.11 -m uv venv
source .venv/bin/activate
```

2. move `src/*.py`, param.yaml, requirements.txt, data/data.xml to this repo from [example repo](https://github.com/iterative/example-get-started-experiments/tree/main)

3. install the requirements.txt

```
uv pip install -r requirements.txt 
```

4. check if the workflow runnable:

```
python src/data_split.py
python src/train.py
python src/evaluate.py
```

## Step2: add experiment stages to DVC

1. init dvc:
```
dvc init
```

2. add first stage (`data_split`):
```
dvc stage add --name data_split \
  --params base,data_split \
  --deps data/pool_data --deps src/data_split.py \
  --outs data/train_data --outs data/test_data \
  python src/data_split.py
```
3. add dvc control to git
```
git add dvc.yaml data/.gitignore
dvc config core.autostage true
```

4. add second stage (`train`):

```
dvc stage add -n train \
  -p base,train \
  -d src/train.py -d data/train_data \
  -o models/model.pkl \
  python src/train.py
```