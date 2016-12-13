---
title: tmp
date: 2016-12-12 10:24:19
tags:
---


```
sudo easy_install pip
sudo pip install --upgrade virtualenv

mkdir -p ~/venvs/tensorflow
virtualenv --system-site-packages ~/venvs/tensorflow
source ~/venvs/tensorflow/bin/activate


export TF_BINARY_URL=https://storage.googleapis.com/tensorflow/mac/cpu/tensorflow-0.12.0rc0-py2-none-any.whl
pip install --upgrade $TF_BINARY_URL


deactivate

```






