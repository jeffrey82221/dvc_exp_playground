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

5. add third stage (`evaluate`)

```
dvc stage add -n evaluate \
  -p base,evaluate \
  -d src/evaluate.py -d models/model.pkl -d data/test_data \
  -o results python src/evaluate.py
```


## Step3: conduct experiments

1. conduct experiment with assigned parameters 

```
dvc exp run --set-param "train.img_size=128"
```
```
git commit -m 'experiment: first run'
```
```
dvc exp run -S "train.img_size=384"
```
```
dvc metrics diff
```

>> 
Path                           Metric      HEAD     workspace    Change
results/evaluate/metrics.json  dice_multi  0.86208  0.87073      0.00865

2. hyperparameter tuning 

```
dvc exp run --name "arch-size" --queue \
-S 'train.arch=alexnet,resnet34,squeezenet1_1' \
-S 'train.img_size=128,256'
```
(This is a grid search)
>>>

Queueing with overrides '{'params.yaml': ['train.arch=alexnet', 'train.img_size=128']}'.
Queued experiment 'arch-size-1' for future execution.
Queueing with overrides '{'params.yaml': ['train.arch=alexnet', 'train.img_size=256']}'.
Queued experiment 'arch-size-2' for future execution.
Queueing with overrides '{'params.yaml': ['train.arch=resnet34', 'train.img_size=128']}'.
Queued experiment 'arch-size-3' for future execution.
Queueing with overrides '{'params.yaml': ['train.arch=resnet34', 'train.img_size=256']}'.
Queued experiment 'arch-size-4' for future execution.
Queueing with overrides '{'params.yaml': ['train.arch=squeezenet1_1', 'train.img_size=128']}'.
Queued experiment 'arch-size-5' for future execution.
Queueing with overrides '{'params.yaml': ['train.arch=squeezenet1_1', 'train.img_size=256']}'.
Queued experiment 'arch-size-6' for future execution.

```
dvc queue status
```
>>>
Task     Name         Created    Status
0d4ee41  arch-size-1  02:02 PM   Queued
5b2e53a  arch-size-2  02:02 PM   Queued
632daee  arch-size-3  02:02 PM   Queued
7ee2b79  arch-size-4  02:02 PM   Queued
73aa7be  arch-size-5  02:02 PM   Queued
4e4a5ca  arch-size-6  02:02 PM   Queued

Worker status: 0 active, 0 idle
```
dvc queue start -j 6
```