# Introduction

This's the tool for CTR, including FM, FFM, NFFM, XdeepFM and so on. 

***Note: only implement FM, more detail and another models will be implemented***



# Requirements

- python3
- TensorFlow>=1.4



#  Quick Start

## Loading dataset

```python
import ctrNet
import pandas as pd
import numpy as np
import tensorflow as tf
import ctrNet
from src import hparams
from sklearn.model_selection import train_test_split
from src import misc_utils as utils
import os
train_df=pd.read_csv('data/train_small.txt',header=None,sep='\t')
train_df.columns=['label']+['f'+str(i) for i in range(39)]
train_df, dev_df,_,_ = train_test_split(train_df,train_df,test_size=0.1, random_state=2019)
dev_df, test_df,_,_ = train_test_split(dev_df,dev_df,test_size=0.5, random_state=2019)
features=['f'+str(i) for i in range(39)]
```

##  Createing hparams

```python
hparam=tf.contrib.training.HParams(
            model='fm',
            k=16,
            hash_ids=int(1e6),
            batch_size=64,
            optimizer="adam",
            learning_rate=0.0002,
            num_display_steps=100,
            num_eval_steps=1000,
            epoch=3,
            metric='auc',
            init_method='uniform',
            init_value=0.1,
            feature_nums=len(features))
utils.print_hparams(hparam)
```

##  Building model

```python
os.environ["CUDA_DEVICE_ORDER"]='PCI_BUS_ID'
os.environ["CUDA_VISIBLE_DEVICES"]='0'
model=ctrNet.build_model(hparam)
```

## Training model

```python
#You can use control-c to stop training if the model doesn't improve.
model.train(train_data=(train_df[features],train_df['label']),\
            dev_data=(dev_df[features],dev_df['label']))
```

## Testing model

```python
from sklearn import metrics
preds=model.infer(dev_data=(test_df[features],test_df['label']))
fpr, tpr, thresholds = metrics.roc_curve(test_df['label']+1, preds, pos_label=2)
auc=metrics.auc(fpr, tpr)
print(auc)
```



