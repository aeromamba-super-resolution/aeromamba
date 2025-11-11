# AEROMamba

## About 

Official PyTorch implementation of 

**AEROMamba: An efficient architecture for audio super-resolution using generative adversarial networks and state space models**

whose demo is available in our [Webpage](https://aeromamba-super-resolution.github.io/).  Our model is closely related to [AERO](https://github.com/slp-rl/aero) and [Mamba](https://github.com/state-spaces/mamba), so make sure to check them out if any questions arise regarding these modules.

## Installation

Requirements:
- Python 3.10.0
- Pytorch 1.12.1
- CUDA 11.3

Instructions:
- Create a conda environment `conda create -n <name_ev> python=3.10` or venv with python==3.10.0 
- Run `pip install -r requirements.txt`

If there is any error in the previous step, make sure to install manually the required libs. For PyTorch/CUDA and Mamba, manual installation is done through 

- `CAUSAL_CONV1D_FORCE_BUILD=TRUE CAUSAL_CONV1D_SKIP_CUDA_BUILD=TRUE CAUSAL_CONV1D_FORCE_CXX11_ABI=TRUE pip install causal_conv1d==1.1.2.post1`
- `CAUSAL_CONV1D_FORCE_BUILD=TRUE CAUSAL_CONV1D_SKIP_CUDA_BUILD=TRUE CAUSAL_CONV1D_FORCE_CXX11_ABI=TRUE pip install mamba-ssm==1.1.3.post1`
- `conda install pytorch==1.12.1 torchvision==0.13.1 torchaudio==0.12.1 cudatoolkit=11.3 -c pytorch`

Also, make sure to unzip the contents of [Mamba](https://github.com/state-spaces/mamba/archive/refs/tags/v1.1.3.post1.zip) (the mamba folder) inside aeromamba/src/models/ .

### ViSQOL

We did not use ViSQOL for training and validation, but if you want to, see [AERO](https://github.com/slp-rl/aero) for instructions. 

## Datasets

### Download data

For popular music we use the mixture tracks of [MUSDB18-HQ](https://sigsep.github.io/datasets/musdb.html#musdb18-hq-uncompressed-wav) dataset.

For piano music, we collected a private dataset from CDs whose metadata are described in our [Webpage](https://aeromamba-super-resolution.github.io/).

### Resample data

Data are a collection of high/low resolution pairs. Corresponding high and low resolution signals should be in different folders, eg: hr_dataset and lr_dataset. 

In order to create each folder, one should run `resample_data` a total of 5 times,
to include all source/target pairs.

We downsample once to a target 11.025 kHz, from the original 44.1 kHz.

e.g. for 11.025 and 44.1 kHz: \
`python data_prep/resample_data.py --data_dir <path for 44.1 kHz data> --out_dir <path for 11.025 kHz data> --target_sr 11025`

### Create egs files

For each low and high resolution pair, one should create "egs files" twice: for low and high resolution.  
`create_meta_files.py` creates a pair of train and val "egs files", each under its respective folder.
Each "egs file" contains meta information about the signals: paths and signal lengths.

`python data_prep/create_meta_files.py <path for 11.025 kHz data> egs/musdb/ lr` 

`python data_prep/create_meta_files.py <path for 44.1 kHz data> egs/musdb/ hr`

## Train

Run `train.py` with `dset` and `experiment` parameters, or set the default values in main_config.yaml file.  

`
python train.py dset=<dset-name> experiment=<experiment-name>
`

To train with multiple GPUs, run with parameter `ddp=true`. e.g.
`
python train.py dset=<dset-name> experiment=<experiment-name> ddp=true
`

## Test (on whole dataset)

`
python test.py dset=<dset-name> experiment=<experiment-name>
`

## Inference

### Single sample

`
python predict.py dset=<dset-name> experiment=<experiment-name> +filename=<absolute path to input file> +output=<absolute path to output directory>
`

### Multiple samples

`
bash predict_batch.sh <input_folder> <output_folder>
`

We also provide predict_with_ola.py to predict large files that do not fit in the GPU, without the need for segmentation, using Overlap-and-Add. The original predict.py is also capable of joining predicted segments, but its naïve method causes clicks. 

`
python predict_with_ola.py dset=<dset-name> experiment=<experiment-name> +folder_path=<absolute path to input folder> +output=<absolute path to output directory>
`
### Checkpoints

To use pre-trained models for MUSDB18-HQ or PianoEval data, one can download checkpoints from [here](https://poliufrjbr-my.sharepoint.com/:f:/g/personal/abreu_engcb_poli_ufrj_br/EhqOtFGTmeZNr-WNv976Jw8BLfpgBYisodrRb2uTGvrFsg?e=5j1nx4).

To link to checkpoint when testing or predicting, override/set path under `checkpoint_file:<path>` in `conf/main_config.yaml.` e.g.

`
python test.py dset=<dset-name> experiment=<experiment-name> +checkpoint_file=<path to checkpoint.th file>
`

Alternatively, make sure that the checkpoint file is in its corresponding output folder:  
For each low to high resolution setting, hydra creates a folder under `outputs/<dset-name>/<experiment-name>`

Make sure that `restart: false` in `conf/main_config.yaml`

### Citation

@inproceedings{Abreu2024lamir,
        author    = {Wallace Abreu and Luiz Wagner Pereira Biscainho},
        title     = {AEROMamba: An Efficient Architecture for Audio Super-Resolution Using Generative Adversarial Networks and State Space Models},
        booktitle = {Proceedings of the 1st Latin American Music Information Retrieval Workshop},
        year      = {2024},
        address   = {Rio de Janeiro, Brazil},
      }
      
