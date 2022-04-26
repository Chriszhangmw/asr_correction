# Adapter-based Cross-lingual ASR with EasyEspnet

This is a Adapter-based cross-lingual ASR implementation of our works [1, 2] built on top of [EasyEspnet](https://github.com/jindongwang/EasyEspnet). Please refer to it for the basic introduction, installation and usage.

## Run

After extracting features using Espnet and set the data folder path as introduced in [EasyEspnet](https://github.com/jindongwang/EasyEspnet).

You need to check or modify in `train.py arg_list`, config should be in ESPnet config style (remember to include decoding information if you want to compute cer/wer), then, you can run train.py. For example, 

```
python train.py --root_path commonvoice/asr1 --dataset ar --config config/adapter_example.yaml
```

dataset here refers to the language code used in the Common Voice corpus. Results (log, model, snapshots) are saved in results_(dataset)/(config_name) by default.

### Adapters

Adapters is a parameter-efficient way for cross-lingual ASR as introduced in [1, 2]. Given a pre-trained multilingual ASR model, the adapters are injected into the model, during adaptation, only the adapters and language-specific heads are updated, while the main body of the model is frozen, resulting in much faster training speed and making the adaptation more stable. Please refer to our SimAdapter paper [2] for details.

To train the adapters, generally you need to specify the `load_pretrained_model` in the config files to load the multilingual ASR model. We found that by splitting the training of language-specific heads and adapters, the adaptation performance can be further improved [2]. You may also train the language-specific heads first and set `train_adapter_with_head` to `false` during adapters' training.

For the vanilla Adapter method, we provide an example config in `config/adapter_example.yaml`, please refer to it for modification to train your own adapter.

For the Meta-Adapter, there are two stages, the first step is pre-training, we provide an example config in `config/meta_adapter_example.yaml`, please refer to it for modification to train your own meta-adapter; the second stage is fine-tuning, please refer to the `config/finetune_meta_adapter_example.yaml`. Please refer to our meta-adapter paper [1] for details.

### SimAdapter

To leverage the information from multiple source / target adapters, we introduce SimAdapter for cross-lingual ASR [2]. To train a SimAdapter, you need a model with multiple adapters/meta-adapters injected and pre-trained. During SimAdapter, we initialize attention layers after every adapter outputs to fuse the information from multiple adapters (languages). During training of SimAdapter, parameters of the main body, the adapters, and the language heads are all frozen. Only the SimAdapter's fusion layers are trained. Please refer to our SimAdapter paper [2] for details.

We also provide a template config for you to perform SimAdapter: `config/simadapter_example.yaml`

### Demo

TBD

## Decoding and WER/CER evaluation

Set `--decoding_mode` to `true` to perform decoding and CER/WER evaluation. For example:

```
python train.py --root_path commonvoice/asr1 --dataset ar --decoding_mode true --config config/adapter_example.yaml
```

## Distributed training

Following EasyEspnet, you can also perform distributed training which is much faster. For example, using 4 GPUs, 1 node: 

```
CUDA_VISIBLE_DEVICES=0,1,2,3 python -m torch.distributed.launch --nproc_per_node=4 train.py --dist_train true --root_path commonvoice/asr1 --dataset ar --config config/adapter_example.yaml
```


## Contact

- [Wenxin Hou](https://houwx.net/): houwx001@gmail.com
- [Jindong Wang](http://www.jd92.wang/): jindongwang@outlook.com


## Code of Conduct
This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct),
[trademark notice](https://docs.opensource.microsoft.com/releasing/), and [security reporting instructions](https://docs.opensource.microsoft.com/releasing/maintain/security/).


## References

[1] Wenxin Hou, Yidong Wang, Shengzhou Gao, Takahiro Shinozaki, “Meta-Adapter: Efficient Cross-Lingual Adaptation with Meta-Learning”, in Proc. IEEE International Conference on Acoustics, Speech, and Signal Processing (ICASSP), Toronto, Ontario, Canada, June 2021. [[paper]](https://ieeexplore.ieee.org/document/9414959)

[2] Wenxin Hou, Han Zhu, Yidong Wang, Jindong Wang, Tao Qin, Renjun Xu, Takahiro Shinozaki, ”Exploiting Adapters for Cross-lingual Low-resource Speech Recognition”, Arxiv preprint. [[paper]](https://arxiv.org/abs/2105.11905)