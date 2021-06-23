# Docuwarp
[![Codacy Badge](https://api.codacy.com/project/badge/Grade/7acd5aa8048a4c96bd97a96bac2639d1)](https://app.codacy.com/app/huang836/deep-learning-for-document-dewarping?utm_source=github.com&utm_medium=referral&utm_content=thomasjhuang/deep-learning-for-document-dewarping&utm_campaign=Badge_Grade_Dashboard)
![Python version](https://img.shields.io/pypi/pyversions/dominate.svg?style=flat)

This project is focused on dewarping document images through the usage of pix2pixHD, a GAN that is useful for general image to image translation. The objective is to take images of documents that are warped, folded, crumpled, etc. and convert the image to a "dewarped" state by using [pix2pixHD](https://github.com/NVIDIA/pix2pixHD) to train and perform inference. All of the model code is borrowed directly from the pix2pixHD official repository.

Some of the intuition behind doing this is inspired by these two papers:
1. [DocUNet: Document Image Unwarping via A Stacked U-Net (Ma et.al)](https://www.juew.org/publication/DocUNet.pdf)
2. [Document Image Dewarping using Deep Learning (Ramanna et.al)](www.insticc.org/Primoris/Resources/PaperPdf.ashx?idPaper=73684)

## Prerequisites

This project requires **Python** and the following Python libraries installed:

-   Linux or OSX
-   [scikit-learn](http://scikit-learn.org/stable/)
-   NVIDIA GPU (11G memory or larger) + CUDA cuDNN
-   [Pytorch](https://pytorch.org/get-started/locally/)
-   [Pillow](https://pillow.readthedocs.io/en/stable/installation.html)
-   [OpenCV](https://opencv-python-tutroals.readthedocs.io/en/latest/py_tutorials/py_setup/py_table_of_contents_setup/py_table_of_contents_setup.html)

## Getting Started
### Installation
-   Install PyTorch and dependencies from <http://pytorch.org>
-   Install python libraries [dominate](https://github.com/Knio/dominate).
```bash
pip install dominate
```
-   Clone this repo:
```bash
git clone https://github.com/thomasjhuang/deep-learning-for-document-dewarping
cd deep-learning-for-document-dewarping
```

### Training
-   Train the kaggle model with 256x256 crops:
```bash
python train.py --name kaggle --label_nc 0 --no_instance --no_flip --netG local --ngf 32 --fineSize 256
```
-   To view training results, please checkout intermediate results in `./checkpoints/kaggle/web/index.html`.
If you have tensorflow installed, you can see tensorboard logs in `./checkpoints/kaggle/logs` by adding `--tf_log` to the training scripts.

### Training with your own dataset
-   If you want to train with your own dataset, please generate label maps which are one-channel whose pixel values correspond to the object labels (i.e. 0,1,...,N-1, where N is the number of labels). This is because we need to generate one-hot vectors from the label maps. Please also specity `--label_nc N` during both training and testing.
-   If your input is not a label map, please just specify `--label_nc 0` which will directly use the RGB colors as input. The folders should then be named `train_A`, `train_B` instead of `train_label`, `train_img`, where the goal is to translate images from A to B.
-   If you don't have instance maps or don't want to use them, please specify `--no_instance`.
-   The default setting for preprocessing is `scale_width`, which will scale the width of all training images to `opt.loadSize` (1024) while keeping the aspect ratio. If you want a different setting, please change it by using the `--resize_or_crop` option. For example, `scale_width_and_crop` first resizes the image to have width `opt.loadSize` and then does random cropping of size `(opt.fineSize, opt.fineSize)`. `crop` skips the resizing step and only performs random cropping. If you don't want any preprocessing, please specify `none`, which will do nothing other than making sure the image is divisible by 32.

### Testing
-   Test the model:
```bash
python test.py --name kaggle --label_nc 0 --netG local --ngf 32 --resize_or_crop crop --no_instance --no_flip --fineSize 256
```
The test results will be saved to a directory here: `./results/kaggle/test_latest/`.


### Dataset
-   I use the kaggle denoising dirty documents dataset. To train a model on the full dataset, please download it from the [official website](https://www.kaggle.com/c/denoising-dirty-documents/data).
After downloading, please put it under the `datasets` folder with warped images under the directory name `train_A` and unwarped images under the directory `train_B`. Your test images are warped images, and should be under the name `test_A`. Below is an example dataset directory structure.

            .
            ├── ...
            ├── datasets                  
            │   ├── train_A               # warped images
            │   ├── train_B               # unwarped, "ground truth" images
            │   └── test_A                # warped images used for testing
            └── ...
     
### Multi-GPU training
-   Train a model using multiple GPUs (`bash ./scripts/train_kaggle_256_multigpu.sh`):
```bash
#!./scripts/train_kaggle_256_multigpu.sh
python train.py --name kaggle_256_multigpu --label_nc 0 --netG local --ngf 32 --resize_or_crop crop --no_instance --no_flip --fineSize 256 --batchSize 32 --gpu_ids 0,1,2,3,4,5,6,7
```

### Training with Automatic Mixed Precision (AMP) for faster speed
-   To train with mixed precision support, please first install apex from: <https://github.com/NVIDIA/apex>
-   You can then train the model by adding `--fp16`. For example,
```bash
#!./scripts/train_512p_fp16.sh
python -m torch.distributed.launch train.py --name label2city_512p --fp16
```
In my test case, it trains about 80% faster with AMP on a Volta machine.

## More Training/Test Details
-   Flags: see `options/train_options.py` and `options/base_options.py` for all the training flags; see `options/test_options.py` and `options/base_options.py` for all the test flags.
-   Instance map: we take in both label maps and instance maps as input. If you don't want to use instance maps, please specify the flag `--no_instance`.
