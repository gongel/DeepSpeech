# Multi Band MelGAN with CSMSC
This example contains code used to train a [Multi Band MelGAN](https://arxiv.org/abs/2005.05106) model with [Chinese Standard Mandarin Speech Copus](https://www.data-baker.com/open_source.html).
## Dataset
### Download and Extract the datasaet
Download CSMSC from the [official website](https://www.data-baker.com/data/index/source) and extract it to `~/datasets`. Then the dataset is in directory `~/datasets/BZNSYP`.

### Get MFA results for silence trim
We use [MFA](https://github.com/MontrealCorpusTools/Montreal-Forced-Aligner) results to  cut silence in the edge of audio.
You can download from here [baker_alignment_tone.tar.gz](https://paddlespeech.bj.bcebos.com/MFA/BZNSYP/with_tone/baker_alignment_tone.tar.gz), or train your own MFA model reference to [use_mfa example](https://github.com/PaddlePaddle/Parakeet/tree/develop/examples/use_mfa) of our repo.

## Get Started
Assume the path to the dataset is `~/datasets/BZNSYP`.
Assume the path to the MFA result of CSMSC is `./baker_alignment_tone`.
Run the command below to
1. **source path**.
2. preprocess the dataset,
3. train the model.
4. synthesize wavs.
    - synthesize waveform from `metadata.jsonl`.
```bash
./run.sh
```
### Preprocess the dataset
```bash
./local/preprocess.sh ${conf_path}
```
When it is done. A `dump` folder is created in the current directory. The structure of the dump folder is listed below.

```text
dump
├── dev
│   ├── norm
│   └── raw
├── test
│   ├── norm
│   └── raw
└── train
    ├── norm
    ├── raw
    └── feats_stats.npy
```
The dataset is split into 3 parts, namely `train`, `dev` and `test`, each of which contains a `norm` and `raw` subfolder. The `raw` folder contains log magnitude of mel spectrogram of each utterances, while the norm folder contains normalized spectrogram. The statistics used to normalize the spectrogram is computed from the training set, which is located in `dump/train/feats_stats.npy`.

Also there is a `metadata.jsonl` in each subfolder. It is a table-like file which contains id and paths to spectrogam of each utterance.

### Train the model
```bash
CUDA_VISIBLE_DEVICES=${gpus} ./local/train.sh ${conf_path} ${train_output_path}
```
`./local/train.sh` calls `${BIN_DIR}/train.py`.
Here's the complete help message.

```text
usage: train.py [-h] [--config CONFIG] [--train-metadata TRAIN_METADATA]
                [--dev-metadata DEV_METADATA] [--output-dir OUTPUT_DIR]
                [--ngpu NGPU] [--verbose VERBOSE]

Train a Multi-Band MelGAN model.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG       config file to overwrite default config.
  --train-metadata TRAIN_METADATA
                        training data.
  --dev-metadata DEV_METADATA
                        dev data.
  --output-dir OUTPUT_DIR
                        output dir.
  --ngpu NGPU           if ngpu == 0, use cpu.
  --verbose VERBOSE     verbose.
```

1. `--config` is a config file in yaml format to overwrite the default config, which can be found at `conf/default.yaml`.
2. `--train-metadata` and `--dev-metadata` should be the metadata file in the normalized subfolder of `train` and `dev` in the `dump` folder.
3. `--output-dir` is the directory to save the results of the experiment. Checkpoints are save in `checkpoints/` inside this directory.
4. `--ngpu` is the number of gpus to use, if ngpu == 0, use cpu.

### Synthesize
`./local/synthesize.sh` calls `${BIN_DIR}/synthesize.py`, which can synthesize waveform from `metadata.jsonl`.
```bash
CUDA_VISIBLE_DEVICES=${gpus} ./local/synthesize.sh ${conf_path} ${train_output_path} ${ckpt_name}
```
```text
usage: synthesize.py [-h] [--config CONFIG] [--checkpoint CHECKPOINT]
                     [--test-metadata TEST_METADATA] [--output-dir OUTPUT_DIR]
                     [--ngpu NGPU] [--verbose VERBOSE]

Synthesize with multi band melgan.

optional arguments:
  -h, --help            show this help message and exit
  --config CONFIG       multi band melgan config file.
  --checkpoint CHECKPOINT
                        snapshot to load.
  --test-metadata TEST_METADATA
                        dev data.
  --output-dir OUTPUT_DIR
                        output dir.
  --ngpu NGPU           if ngpu == 0, use cpu.
  --verbose VERBOSE     verbose.
```

1. `--config` multi band melgan config file. You should use the same config with which the model is trained.
2. `--checkpoint` is the checkpoint to load. Pick one of the checkpoints from `checkpoints` inside the training output directory.
3. `--test-metadata` is the metadata of the test dataset. Use the `metadata.jsonl` in the `dev/norm` subfolder from the processed directory.
4. `--output-dir` is the directory to save the synthesized audio files.
5. `--ngpu` is the number of gpus to use, if ngpu == 0, use cpu.

## Pretrained Models
Pretrained model can be downloaded here [mb_melgan_baker_ckpt_0.5.zip](https://paddlespeech.bj.bcebos.com/Parakeet/mb_melgan_baker_ckpt_0.5.zip).
Static model can be downloaded here [mb_melgan_baker_static_0.5.zip](https://paddlespeech.bj.bcebos.com/Parakeet/mb_melgan_baker_static_0.5.zip)

Multi Band MelGAN checkpoint contains files listed below.

```text
mb_melgan_baker_ckpt_0.5
├── default.yaml                  # default config used to train multi band melgan
├── feats_stats.npy               # statistics used to normalize spectrogram when training multi band melgan
└── snapshot_iter_1000000.pdz     # generator parameters of multi band melgan
```
## Acknowledgement
We adapted some code from https://github.com/kan-bayashi/ParallelWaveGAN.
