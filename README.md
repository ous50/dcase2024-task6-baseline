# dcase2024-task6-baseline

<div align="center">

**DCASE2024 Challenge Task 6 baseline system of Automated Audio Captioning (AAC)**

<a href="https://www.python.org/">
    <img alt="Python" src="https://img.shields.io/badge/-Python 3.11-blue?style=for-the-badge&logo=python&logoColor=white">
</a>
<a href="https://pytorch.org/get-started/locally/">
    <img alt="PyTorch" src="https://img.shields.io/badge/-PyTorch 2.2-ee4c2c?style=for-the-badge&logo=pytorch&logoColor=white">
</a>
<a href="https://black.readthedocs.io/en/stable/">
    <img alt="Code style: black" src="https://img.shields.io/badge/code%20style-black-black.svg?style=for-the-badge&labelColor=gray">
</a>
<a href="https://github.com/Labbeti/dcase2024-task6-baseline/actions">
    <img alt="Build" src="https://img.shields.io/github/actions/workflow/status/Labbeti/dcase2024-task6-baseline/test.yaml?branch=main&style=for-the-badge&logo=github">
</a>

</div>

The main model is composed of a pretrained convolutional encoder to extract features and a transformer decoder to generate caption.
For more information, please refer to the corresponding [DCASE task page](https://dcase.community/challenge2024/task-automated-audio-captioning).

**This repository includes:**
- AAC model trained on the **Clotho** dataset
- Extract features using **ConvNeXt**
- System reaches **29.6% SPIDEr-FL** score on Clotho-eval (development-testing)
- Output detailed training characteristics (number of parameters, MACs, energy consumption...)


## Installation
First, you need to create an environment that contains **python>=3.11** and **pip**. You can use venv, conda, micromamba or other python environment tool.

Here is an example with [micromamba](https://mamba.readthedocs.io/en/latest/user_guide/micromamba.html):
```bash
micromamba env create -n env_dcase24 python=3.11 pip -c defaults
micromamba activate env_dcase24
```

Then, you can clone this repository and install it:
```bash
git clone https://github.com/ous50/dcase2024-task6-baseline
cd dcase2024-task6-baseline
pip install -e .
pre-commit install
```

> [!IMPORTANT]
> After a whole year, the Pypi index is updated and pip will not install the package correctly. I have updated requirements to try to fix this issue. If you encounter any problem, please install `Pypi-timemachine` and run a timeed-back server with the following command:
> ```bash
> pip install pypi-timemachine
> pypi-timemachine 2024-04-20 --port 11451
> ```
> You would have to keep this command running in a terminal, either in a separate terminal or in the background.
> Then, you can install the package with the following command:
> ```bash
> pip install -e . --index-url http://localhost:11451
> pre-commit install
> ```

> [!TIP]
> If you are renting a cloud GPU instance, be sure to choose a GPU with at least 11GB of VRAM. The model requires approximatively 10GB of VRAM to train.
> The server location should be in Europe to download the Clotho dataset. If you are in another region, you can change the server location by setting the environment variable `AAC_DATASETS_SERVER` to `https://datasets.aac.eurecom.fr`.
> Otherwise, you can download the dataset manually and put it in the `./data` directory.
> It is recommanded to use a download tool that supports resuming downloads like `wget`, `curl` or `aria2`.
> The dataset link is available on the [Clotho dataset page](https://clotho-dataset.github.io/).
> Or you can use the following links:
> 
> https://zenodo.org/record/4783391/files/clotho_captions_development.csv
> https://zenodo.org/record/4783391/files/clotho_metadata_development.csv
> https://zenodo.org/record/4783391/files/clotho_captions_validation.csv
> https://zenodo.org/record/4783391/files/clotho_metadata_validation.csv
> https://zenodo.org/record/4783391/files/clotho_captions_evaluation.csv
> https://zenodo.org/record/4783391/files/clotho_metadata_evaluation.csv
> https://zenodo.org/record/3865658/files/clotho_metadata_test.csv
>
> https://zenodo.org/record/4783391/files/clotho_audio_development.7z
> https://zenodo.org/record/4783391/files/clotho_audio_validation.7z
> https://zenodo.org/record/4783391/files/clotho_audio_evaluation.7z
> https://zenodo.org/record/3865658/files/clotho_audio_test.7z
> https://zenodo.org/record/6610709/files/clotho_analysis_2022.zip
>
> ```bash
> mkdir -p data
> wget -P data -i clotho_dataset_link.txt
> ```
>

You also need to install Java >= 1.8 and <= 1.13 on your machine to compute AAC metrics. If needed, you can override java executable path with the environment variable `AAC_METRICS_JAVA_PATH`.

> [!WARNING]
> It is known that OpenJDK 8 has a bug that prevents the AAC metrics from running. You can use [AzulJDK 13](https://www.azul.com/downloads/?version=java-13&show-old-builds=true#zulu), which is tested by [ous](https://github.com/ous50), to run the metrics.
> 
> For Ubuntu or Apt based system, You can also use the following command to install AzulJDK 13:
> ```bash
> sudo apt install gnupg ca-certificates curl
> curl -s https://repos.azul.com/azul-repo.key | sudo gpg --dearmor -o /usr/share/keyrings/azul.gpg
> echo "deb [signed-by=/usr/share/keyrings/azul.gpg] https://repos.azul.com/zulu/deb stable main" | sudo tee /etc/apt/sources.list.d/zulu.list
> sudo apt update
> sudo apt install zulu13-jdk
> ```
> Then, you can set the environment variable `AAC_METRICS_JAVA_PATH` to `/usr/lib/jvm/zulu13/bin/java`.




## Usage

### Download external data, models and prepare

To download, extract and process data, you need to run:
```bash
dcase24t6-prepare
```
By default, the dataset is stored in `./data` directory. It will requires approximatively 33GB of disk space.

### Train the default model

```bash
dcase24t6-train +expt=baseline
```

By default, the model and results are saved in directory `./logs/SAVE_NAME`. `SAVE_NAME` is the name of the script with the starting date.
Metrics are computed at the end of the training with the best checkpoint.

### Test a pretrained model

```bash
dcase24t6-test resume=./logs/SAVE_NAME
```
or specify each path separtely:
```bash
dcase24t6-test resume=null model.checkpoint_path=./logs/SAVE_NAME/checkpoints/MODEL.ckpt tokenizer.path=./logs/SAVE_NAME/tokenizer.json
```
You need to replace `SAVE_NAME` by the save directory name and `MODEL` by the checkpoint filename.

If you want to load and test the baseline pretrained weights, you can specify the baseline checkpoint weights:

```bash
dcase24t6-test resume=~/.cache/torch/hub/checkpoints/dcase2024-task6-baseline
```

### Inference on a file
If you want to test the baseline model on a single file, you can use the `baseline_pipeline` function:

```python
from dcase24t6.nn.hub import baseline_pipeline

sr = 44100
audio = torch.rand(1, sr * 15)

model = baseline_pipeline()
item = {"audio": audio, "sr": sr}
outputs = model(item)
candidate = outputs["candidates"][0]

print(candidate)
```

## Code overview
The source code extensively use [PyTorch Lightning](https://lightning.ai/docs/pytorch/stable/) for training and [Hydra](https://hydra.cc/) for configuration.
It is highly recommanded to learn about them if you want to understand this code.

Installation has three main steps:
- Download external models ([ConvNeXt](https://github.com/topel/audioset-convnext-inf) to extract audio features)
- Download Clotho dataset using [aac-datasets](https://github.com/Labbeti/aac-datasets)
- Create HDF files containing each Clotho subset with preprocessed audio features using [torchoutil](https://github.com/Labbeti/torchoutil)

Training follows the standard way to create a model with lightning:
- Initialize callbacks, tokenizer, datamodule, model.
- Start fitting the model on the specified datamodule.
- Evaluate the model using [aac-metrics](https://github.com/Labbeti/aac-metrics)


## Model
The model outperforms previous baselines with a SPIDEr-FL score of **29.6%** on the Clotho evaluation subset.
The captioning model architecture is described in [this paper](https://arxiv.org/pdf/2309.00454.pdf) and called **CNext-trans**. The encoder part (ConvNeXt) is described in more detail in [this paper](https://arxiv.org/pdf/2306.00830.pdf).

The pretrained weights of the AAC model are available on Zenodo: [ConvNeXt encoder (BL_AC)](https://zenodo.org/records/8020843), [Transformer decoder](https://zenodo.org/records/10849427). Both weights are automatically downloaded during `dcase24t6-prepare`.

### Main hyperparameters

| Hyperparameter | Value | Option |
| --- | --- | --- |
| Number of epochs | 400 | `trainer.max_epochs` |
| Batch size | 64 | `datamodule.batch_size` |
| Gradient accumulation | 8 | `trainer.accumulate_grad_batches` |
| Learning rate | 5e-4 | `model.lr` |
| Weight decay | 2 | `model.weight_decay` |
| Gradient clipping | 1 | `trainer.gradient_clip_val` |
| Beam size | 3 | `model.beam_size` |
| Model dimension size | 256 | `model.d_model` |
| Label smoothing | 0.2 | `model.label_smoothing` |
| Mixup alpha | 0.4 | `model.mixup_alpha` |


### Detailed results

| Metric | Score on Clotho-eval |
| --- | --- |
| BLEU-1 | 0.5948 |
| BLEU-2 | 0.3924 |
| BLEU-3 | 0.2603 |
| BLEU-4 | 0.1695 |
| METEOR | 0.1897 |
| ROUGE-L | 0.3927 |
| CIDEr-D | 0.4619 |
| SPICE | 0.1335 |
| SPIDEr | 0.2977 |
| SPIDEr-FL | 0.2962 |
| SBERT-sim | 0.5059 |
| FER | 0.0038 |
| FENSE | 0.5040 |
| BERTScore | 0.9766 |
| Vocabulary (words) | 551 |

Here is also an estimation of the number of parameters and multiply-accumulate operations (MACs) during inference for the audio file "Santa Motor.wav":

<!--
# encoder:
flops: 89724036608
macs: 44757425184
params: 29388303
duration: 0.030155420303344727

# decoder:
forcing_flops: 471009792
forcing_macs: 235300608
forcing_params: 11911699
forcing_duration: 0.016583681106567383
generate_flops: 5589742080
generate_macs: 2793307392
generate_params: 11911699
generate_duration: 0.14899301528930664
-->

| Name | Params (M) | MACs (G) |
| --- | --- | --- |
| Encoder | 29.4 | 44.4 |
| Decoder | 11.9 | 4.3 |
| Total | 41.3 | 48.8 |

## Tips
- **Modify the model**.
The model class is located in `src/dcase24t6/models/trans_decoder.py`. It is recommanded to create another class and conf to keep different models architectures.
The loss is computed in the method called `training_step`. You can also modify the model architecture in the method called `setup`.

- **Extract different audio features**.
For that, you can add a new pre-process function in `src/dcase24t6/pre_processes` and the related conf in `src/conf/pre_process`. Then, re-run `dcase24t6-prepare pre_process=YOUR_PROCESS download_clotho=false` to create new HDF files with your own features.
To train a new model on these features, you can specify the HDF files required in `dcase24t6-train datamodule.train_hdfs=clotho_dev_YOUR_PROCESS.hdf datamodule.val_hdfs=... datamodule.test_hdfs=... datamodule.predict_hdfs=...`. Depending on the features extracted, some parameters could be modified in the model to handle them.

- **Using as a package**.
If you do not want ot use the entire codebase but only parts of it, you can install it as a package using:

```bash
pip install git+https://github.com/Labbeti/dcase2024-task6-baseline
```

Then you will be able to import any object from the code like for example `from dcase24t6.models.trans_decoder import TransDecoderModel`. There is also several important dependencies that you can install separately:

- `aac-datasets` to download and load AAC datasets,
- `aac-metrics` to compute AAC metrics,
- `torchoutil[extras]` to pack datasets to HDF files.


## Additional information
- The code has been made for **Ubuntu 20.04** and should work on more recent Ubuntu versions and Linux-based distributions.
- The GPU used is **NVIDIA GeForce RTX 2080 Ti** (11GB VRAM). Training lasts for approximatively 2h30m in the default setting.
- In this code, clotho subsets are named according to the **Clotho convention**, not the DCASE convention. See more information [on this page](https://aac-datasets.readthedocs.io/en/stable/data_subsets.html#clotho).


## See also
- [DCASE2023 Audio Captioning baseline](https://github.com/felixgontier/dcase-2023-baseline)
- [DCASE2022 Audio Captioning baseline](https://github.com/felixgontier/dcase-2022-baseline)
- [DCASE2021 Audio Captioning baseline](https://github.com/audio-captioning/dcase-2021-baseline)
- [DCASE2020 Audio Captioning baseline](https://github.com/audio-captioning/dcase-2020-baseline)
- [aac-datasets](https://github.com/Labbeti/aac-datasets)
- [aac-metrics](https://github.com/Labbeti/aac-metrics)


## Contact
Maintainer:
- [Étienne Labbé](https://labbeti.github.io/) "Labbeti": labbeti.pub@gmail.com
- [ous50](https://ous50.moe) "ous": me@ous50.moe

