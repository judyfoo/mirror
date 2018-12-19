# MIRorR: Matchable Image Retrieval by Learning from Surface Reconstruction
Mirror is the matchable image retrieval pipeline for 3D reconstruction and related applications. 
Different from typical object retrieval, matchable image retrieval aims to find similar images with large overlaps.
Typical CNN-based methods do not generalize well to this problem because models are trained to find objects of the same category.
This project proposes a new method to tackle this problem, by utilizing regional feature aggregation and the accurate auto-annotated 3D geometric data.
For more details, you can refer to the paper:

[Matchable Image Retrieval by Learning from Surface Reconstruction](https://arxiv.org/abs/1811.10343)

[Tianwei Shen*](https://home.cse.ust.hk/~tshenaa/), [Zixin Luo*](https://www.cse.ust.hk/~zluoag/), 
[Lei Zhou](https://zlthinker.github.io/), [Runze Zhang](https://home.cse.ust.hk/~rzhangaj/), 
[Siyu Zhu](https://sites.google.com/site/zhusiyucs/), [Tian Fang](https://scholar.google.com.hk/citations?user=CtpU8mUAAAAJ&hl=zh-TW), 
[Long Quan](https://www.cse.ust.hk/~quan/) (* denotes equal contributions)

In [ACCV 2018](http://accv2018.net).

Feel free to submit issues if you have any questions.


## Prerequisites
The code base has been tested under TensorFlow 1.5 (CUDA 8.0) to TensorFlow 1.7 (CUDA 9.0), using Python 2.7.12.

## First Try
Mirror is extremely easy to use, first download the [model files](https://s3-ap-southeast-1.amazonaws.com/awsiostest-deployments-mobilehub-806196172/ACCV2018/model.zip) using the following command, suppose you are in the root directory of the project `$MIRROR_ROOT`
```bash
sh ./data/model/download_models.sh
```

Suppose you have a folder containing the images of the same scene, take [fountain-P11](http://documents.epfl.ch/groups/c/cv/cvlab-unit/www/data/multiview/denseMVS.html) as an example (or any other image folder by your hand), 
generate the image list by `cd` into that folder and output absolute image paths to a file:
```bash
cd /path/to/fountain-p11
ls -d $PWD/*.png > image_list
```

Then generate the .npy features:
```bash
python retrieval/inference.py --img_list /home/tianwei/Data/fountain-p11/image_list --ckpt_step 720000 --img_size 896 --net resnet-50 --pool MAX --rmac_step 1,3,5 --ckpt_path ./data/model/model.ckpt
```

Then output all the absolute paths of feature files (*.npy) to a file and query:
```bash
ls -d $PWD/*.npy > feature_list
python retrieval/deep_query.py --feature_list /home/tianwei/Data/fountain-p11/feature_list --out_dir ./output --top 20 --out_dim 256
```

You should now be able to see a `match_pairs` file in the `output` folder, recording the ranking and corresponding similarity scores (in range [0, 1] where 1 means identical)
between pairs of images (represented by index).
```bash
0 0 1.0
0 1 0.9598151070731027
0 2 0.9315839494977679
0 3 0.9064583369663783
0 4 0.8789794376918247
0 9 0.8777283804757254
0 8 0.868733160836356
0 5 0.8614536830357142
0 10 0.8602364676339286
0 7 0.8555836813790457
0 6 0.8524237496512277
1 1 1.0
1 2 0.9604712350027902
1 0 0.958723885672433
1 3 0.9399954659598214
1 4 0.9106133052280971
... ...
```

This is the basic usage of this repo. If you cannot get it done, please submit a issue in the issue tracker and let us help you! If you find this work useful in your production environment or research paper, please cite
```bash
@inproceedings{shen2018mirror,
    author={Shen, Tianwei and Luo, Zixin and Zhou, Lei and Zhang, Runze and Zhu, Siyu and Fang, Tian and Quan, Long},
    title={Matchable Image Retrieval by Learning from Surface Reconstruction},
    booktitle={The Asian Conference on Computer Vision (ACCV},
    year={2018},
}
```

The following describe our dataset and results presented in the paper. 

## GL3D dataset
![GL3D overview](misc/figures/dataset_view.jpg)

The GL3D contains 90,590 high-resolution images in 378 different scenes. Each scene contains 50 to 1,000 images with large geometric overlaps, covering urban, rural area, or scenic spots captured by drones from multiple scales and perspectives. Several small objects are also included to enrich the data diversity. Each scene data is reconstructed to generate a triangular mesh model by the state-of-the-art 3D reconstruction pipeline.

Currently we release the 9368 test images for evaluation (see #Test). The full dataset can be found [here](https://github.com/lzx551402/GL3D).


## Train
Training code will be released soon.

## Test
Please refer to `pipeline.sh` for using the image retrieval pipeline. We release two trained models to demonstrate the use. 
The googlenet model can be used to reproduce the results in the paper, which achieves 0.758 mAP@200 on GL3D, 0.768 mAP on Oxford5K 
and 0.820 on Paris6K using the default settings in `pipeline.sh`.

We have additionally trained a ResNet-50 model not documented in the original paper, which achieves better performance than GoogleNet on object retrieval datasets (Oxford5K and Paris6K).

 | Model          | GL3D (mAP@200) | Oxford5K           |  Paris6K           |   Holidays  |
 |----------------|----------------|--------------------|--------------------|-------------|
 | GoogleNet      | 0.758          | 0.768              | 0.820              |   0.861     |
 | ResNet-50      | 0.745          | 0.833              | 0.805              |   0.892     |
 | ResNet-50 + QE | -              | 0.894              | 0.858              |   0.881     |

### Note on GL3D
Download the [GL3D test images](https://s3-ap-southeast-1.amazonaws.com/awsiostest-deployments-mobilehub-806196172/GL3D/eval_data.zip) of size `896 x 896` (around 3.7GB) and unzip them to your GL3D image folder (suppose it is `$MIRROR_ROOT/data/gl3d/`), and then run the commands for testing. The detailed instruction is given below.
```bash
# Download evaluation dataset and unzip
wget https://s3-ap-southeast-1.amazonaws.com/awsiostest-deployments-mobilehub-806196172/GL3D/eval_data.zip
mv eval_data.zip ./data/gl3d/
cd data/gl3d/
unzip eval_data.zip
cd -
# Set up testing configs
net_type=resnet-50 #googlenet
pooling=MAX
step=720000 #400000 
ckpt_save_dir=./data/model/model.ckpt 
img_size=896
feature_dim=256
gl3d_root_dir=$MIRROR_ROOT/data/gl3d/
test_image_list=./data/gl3d/eval_img_list.txt
test_feature_list=./data/gl3d/eval_feature_list.txt
rmac_step=1,3,5
output_dir=./output
# Extract features
python retrieval/inference.py --img_list $test_image_list --ckpt_step $step --img_size $img_size \
    --net $net_type --pool $pooling --rmac_step $rmac_step --ckpt_path $ckpt_save_dir --gl3d_root $gl3d_root_dir
# PR-MAC + Top-k
python retrieval/deep_query.py --feature_list $test_feature_list --out_dir $output_dir --top 200 --out_dim $feature_dim --dataset_root $gl3d_root_dir
# Compute statistics
python retrieval/compute_overlap_benchmark.py $test_image_list $output_dir/match_pairs data/gl3d/gl3d_gt_pairs
```

### Note on Oxford5K or Paris6K
To run the model on [Oxford5K](http://www.robots.ox.ac.uk/~vgg/data/oxbuildings/) or [Paris6K](http://www.robots.ox.ac.uk/~vgg/data/parisbuildings/), 
there is one additional step. Suppose you are in the root directory of this repo, you need to first 
compile the C++ program `compute_ap` for computing the average precision (AP).

```bash
cd cpp
g++ -o compute_ap compute_ap.cpp
```

### Note on Holidays
Downloads INRIA Holidays dataset at [official website](http://lear.inrialpes.fr/~jegou/data.php), prepare the required input lists and use the commands in `pipeline.sh` to reproduce the results.

## Related Projects
Also checkout the following related geometric learning repositories:

[GeoDesc](https://github.com/lzx551402/geodesc): Learning Local Descriptors by Integrating Geometry Constraints

[MVSNet](https://github.com/YoYo000/MVSNet): Depth Inference for Unstructured Multi-view Stereo

## License
MIT
