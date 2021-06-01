# D2-Net: A Trainable CNN for Joint Detection and Description of Local Features

This repository contains the implementation of the following paper:

```text
"D2-Net: A Trainable CNN for Joint Detection and Description of Local Features".
M. Dusmanu, I. Rocco, T. Pajdla, M. Pollefeys, J. Sivic, A. Torii, and T. Sattler. CVPR 2019.
```

[Paper on arXiv](https://arxiv.org/abs/1905.03561), [Project page](https://dsmn.ml/publications/d2-net.html)

## dev branch info

List of changes with respect to the main branch:
- **Switch to OpenCV for image resizing and image pyramid**
- **Remove the banning mechanism in multiscale detection** (it was giving priority to worse localized keypoints)
- **Use the soft scores as keypoint scores** (the soft scores should be comparable between pixels unlike the values of activations from different feature maps)
- **Use a soft score threshold at test time** (this should filter out most spurious features, e.g., in the sky or in homogeneous regions)
- **Minor changes to avoid NaNs in training**
- **Removed validation split**
- **Switched to the "curated" version of MegaDepth** (removed scenes with inconsistent depth maps and scenes overlapping with PhotoTourism, c.f. Dusmanu et al., ECCV 2020 and Tyszkiewicz et al., NeurIPS 2020)

The performance is similar to that of the main branch on Aachen Day-Night v1.0:

| Methods | Aachen day | Aachen night | Retrieval |
| - | - | - | - |
| [[main] D2-Net MS](https://www.visuallocalization.net/details/965/) | 85.2 / 92.5 / 97.8 | 86.7 / 89.8 / 98.0 | NetVLAD top 50 |
| [dev] D2-Net MS | 85.7 / 93.4 / 97.5 | 82.7 / 89.8 / 98.0 | NetVLAD top 50 |
| [dev] D2-Net MS dev - 10K | 85.2 / 92.5 / 97.3 | 83.7 / 89.8 / 98.0 | NetVLAD top 50 |
| [dev] [retrain] D2-Net MS | 86.0 / 93.1 / 97.7 | 80.6 / 88.8 / 98.0 | NetVLAD top 50 |
| [dev] [retrain] D2-Net MS dev - 10K | 85.0 / 92.5 / 97.2 | 83.7 / 91.8 / 98.0 | NetVLAD top 50 |
| [dev] [retrain-curated] D2-Net MS | 85.6 / 93.4 / 97.5 | 82.7 / 91.8 / 98.0 | NetVLAD top 50 |
| [dev] [retrain-curated] D2-Net MS dev - 10K | 85.2 / 93.1 / 97.7 | 80.6 / 90.8 / 96.9 | NetVLAD top 50 |

## Getting started

Python 3.6+ is recommended for running our code. [Conda](https://docs.conda.io/en/latest/) can be used to install the required packages:

```bash
conda install pytorch torchvision cudatoolkit=10.0 -c pytorch
conda install h5py imageio imagesize matplotlib numpy scipy tqdm
conda install opencv
```

## Downloading the models

The off-the-shelf **Caffe VGG16** weights and their tuned counterpart can be downloaded by running:

```bash
mkdir models
mkdir models/dev
wget https://dsmn.ml/files/d2-net/d2_ots.pth -O models/d2_ots.pth
wget https://dsmn.ml/files/d2-net/dev/d2.pth -O models/dev/d2_curated.pth
```

## Feature extraction

`extract_features.py` can be used to extract D2 features for a given list of images. The singlescale features require less than 6GB of VRAM for 1200x1600 images. The `--multiscale` flag can be used to extract multiscale features - for this, we recommend at least 12GB of VRAM. 

The output format can be either [`npz`](https://docs.scipy.org/doc/numpy/reference/generated/numpy.savez.html) or `mat`. In either case, the feature files encapsulate three arrays: 

- `keypoints` [`N x 3`] array containing the positions of keypoints `x, y` and the scales `s`. The positions follow the COLMAP format, with the `X` axis pointing to the right and the `Y` axis to the bottom.
- `scores` [`N`] array containing the activations of keypoints (higher is better).
- `descriptors` [`N x 512`] array containing the L2 normalized descriptors.

```bash
python extract_features.py --image_list_file images.txt (--multiscale)
```

## Tuning on MegaDepth

The training pipeline provided here is a PyTorch implementation of the TensorFlow code that was used to train the model available to download above.

### Downloading and preprocessing the MegaDepth dataset

For this part, [COLMAP](https://colmap.github.io/) should be installed. Please refer to the official website for installation instructions.

After downloading the entire [MegaDepth](http://www.cs.cornell.edu/projects/megadepth/) dataset (including SfM models), the first step is generating the undistorted reconstructions. This can be done by calling `undistort_reconstructions.py` as follows:

```bash
python undistort_reconstructions.py --colmap_path /path/to/colmap/executable --base_path /path/to/megadepth
```

Next, `preprocess_megadepth.sh` can be used to retrieve the camera parameters and compute the overlap between images for all scenes. 

```bash
bash preprocess_undistorted_megadepth.sh /path/to/megadepth /path/to/output/folder
```

In case you prefer downloading the undistorted reconstructions and aggregated scene information folder directly, you can find them [here - Google Drive](https://drive.google.com/open?id=1hxpOsqOZefdrba_BqnW490XpNX_LgXPB). You will still need to download the depth maps ("MegaDepth v1 Dataset") from the MegaDepth website.

### Training

After downloading and preprocessing MegaDepth, the training can be started right away:

```bash
python train.py --use_validation --dataset_path /path/to/megadepth --scene_info_path /path/to/preprocessing/output
```

## BibTeX

If you use this code in your project, please cite the following paper:

```bibtex
@InProceedings{Dusmanu2019CVPR,
    author = {Dusmanu, Mihai and Rocco, Ignacio and Pajdla, Tomas and Pollefeys, Marc and Sivic, Josef and Torii, Akihiko and Sattler, Torsten},
    title = {{D2-Net: A Trainable CNN for Joint Detection and Description of Local Features}},
    booktitle = {Proceedings of the 2019 IEEE/CVF Conference on Computer Vision and Pattern Recognition},
    year = {2019},
}
```
