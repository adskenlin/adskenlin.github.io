---
title: Deep Learning on 3D LiDAR Data P1 Dataset
date: 2021-08-16 09:38:16
tags:
- NuScenes
- Deep Learning
- LiDAR
---

# P1 - NuScenes Dataset Introduction

## Why NuScenes?

With increasing inverstment in fields of autonomous driving, tech-companies are creating their own dataset for training their autonomous vehicles. Some of them, like Waymo Dataset, Lyft L5 Dataset and NuScenes Data are widely used in personal uncommercial reasearchs because of their open-source and mutimodal features.

NuScenes Dataset as one of the earliest published dataset in this field contains various kinds of data collected by different sensors such as cameras, radar and LiDARs. It provides toolkit to help researchers easily and quickly get an overview and process the data with provided functions. And Lyft L5 Dataset also uses similiar toolkit.

Thus, let's start with processing NuScenes Dataset. Since we are focusing on deep learning with LiDAR data, which means we have to prepar the LiDAR data for our deep learning neural network, we'll first take an overview of the most important factors of dataset:

+ Size
+ Variety of Scenes
+ Number and Quality of Annotated Objects

|Dataset|Scenes|Size|PCs lidar|Ann.Frames|3D boxes|
|----|----|----|----|----|----|
|NuScenes|1k|5.5hr|400k|40k|1.4M|
|Lyft L5|366|2.5hr|46k|46k|1.3M|
|Waymo Open|1k|5.5hr|200k|200k|12M|

As the table shown above, NuScenes has a larger data size comparing to Lyft L5, and has an easy-to-use toolkit comparing to Waymo Dataset. For training a neural network, the more data we have, the better performance we could achieve. And with more different driving scenes and more different objects classes, the neural network will become robuster after training.

## Toolkit(Devkit) Installation

Devkit provides us various functions to extract the specific data format and data information from dataset. It also contains different matricies for evaluating NN(Neural Network)'s performance with respect to your goals such as prediction, detecton and so on.

+ visit [official website](https://www.nuscenes.org/) and account registration
+ Download Full dataset(v1.0): Trainval and Test Set
+ Devkit Installation (for more information please visit their [github](https://github.com/nutonomy/nuscenes-devkit))

> ```bash
> # under Ubuntu or MacOS
> # if you do not have python 3.6/3.7
> # install python first
> sudo apt install python-pip
> sudo add-apt-repository ppa:deadsnakes/ppa
> sudo apt-get update
> sudo apt-get install python3.7
> sudo apt-get install python3.7-dev
> # then create virtual environment via conda or virtualenvwrapper
> # if you do not have miniconda, google and install it
> # after miniconda installed
> conda create --name deep-learning-nuscenes python=3.7
> conda activate deep-learning-nuscenes
> # to deactivate env, use `source deactivate`
> # then use
> pip install nuscenes-devkit
> ```

+ Verify the installation and follow the tutorials [here](https://www.nuscenes.org/nuscenes?tutorial=nuscenes).

## What could we do with Dev-Kit?

With devkit of nuscenes we could get an overview of the whole dataset and understand the data formats and structure inside the dataset. We would also use the functions, which devkit provides us, in future steps to build a dataset pre-processing pipeline.

As the introduction, let us try some in jupyter notebook to get better understand of the data formats of the dataset.

To receive an overview of the dataset:

```python
from nuscenes.nuscenes import NuScenes
# if you download the mini-dataset
# use version="v1.0-mini"
# dataroot should be the path
# where you store your dataset
nusc = NuScenes(version = "v1.0-trainval", dataroot="/home/ken/Data/Dataset/NuScenes")

# output:
# ======
# Loading NuScenes tables for version v1.0-trainval...
# 23 category,
# 8 attribute,
# 4 visibility,
# 64386 instance,
# 12 sensor,
# 10200 calibrated_sensor,
# 2631083 ego_pose,
# 68 log,
# 850 scene,
# 34149 sample,
# 2631083 sample_data,
# 1166187 sample_annotation,
# 4 map,
# Done loading in 37.807 seconds.
# ======
# Reverse indexing ...
# Done reverse indexing in 6.4 seconds.
# ======
```

As above shown, we could see there are several data types in this dataset, e.g. `category`, `attribute`, `visibility`, `instance`, `sensor`, `calibrated_sensor`, `ego_pose`, `scene`, ... and so on. You can find the official definition [here](https://www.nuscenes.org/nuscenes#data-format).

For our focus, `scene`, `sample`, `sample_data`, `sample_annotation` will be important.

1. scene

   To get information of scenes:

   ```python
   nusc.list_scenes()
   # output:
   # scene-0161, Car overtaking, parking lot, peds, ped ... [18-05-21 15:07:23]   19s, boston-seaport, anns:1970
   # scene-0162, Leaving parking lot, parked cars, hidde... [18-05-21 15:07:43]   19s, boston-seaport, anns:2230
   # ...
   ```

   Watching these outputs, we could know the scenes contain different driving scenarios and each scene lasts about 20 seconds.

   For a specific scene:

   ```python
   my_scene = nusc.scene[0]
   my_scene
   # output:
   # {'token': '73030fb67d3c46cfb5e590168088ae39',
   # 'log_token': '6b6513e6c8384cec88775cae30b78c0e',
   # 'nbr_samples': 40,
   # 'first_sample_token': 'e93e98b63d3b40209056d129dc53ceee',
   # 'last_sample_token': '40e413c922184255a94f08d3c10037e0',
   # 'name': 'scene-0001',
   # 'description': 'Construction, maneuver between several trucks'}
   
   # explanation:
   # 'token' -- Code for searching and calling this scene
   # 'log_token' -- Code for searching and calling this scene's log
   # 'nbr_samples' -- number of samples in this scene
   # 'first_sample_token/last_sample_token' -- Code for searching and calling this scene's first/last frame
   # 'name', 'decription' -- other information
   ```

2. sample

   A sample means a frame. In this dataset, the data is collected every 0.1 second(10 Hz), and it is annotated every half a second(2 Hz), which means every 5 frames, we would have 1 annotated sample. In annotated sample, the existed objects will be annotated.

   Let us take a sample as example:

   ```python
   first_sample_token = my_scene['first_sample_token']
   my_sample = nusc.get('sample', first_sample_token)
   my_sample

   # output:
   # {'token': 'e93e98b63d3b40209056d129dc53ceee',
   # 'timestamp': 1531883530449377,
   # 'prev': '',
   # 'next': '14d5adfe50bb4445bc3aa5fe607691a8',
   # 'scene_token': '73030fb67d3c46cfb5e590168088ae39',
   # 'data': {'RADAR_FRONT': 'bddd80ae33ec4e32b27fdb3c1160a30e',
   # 'RADAR_FRONT_LEFT': '1a08aec0958e42ebb37d26612a2cfc57',
   # 'RADAR_FRONT_RIGHT': '282fa8d7a3f34b68b56fb1e22e697668',
   # 'RADAR_BACK_LEFT': '05fc4678025246f3adf8e9b8a0a0b13b',
   # 'RADAR_BACK_RIGHT': '31b8099fb1c44c6381c3c71b335750bb',
   # 'LIDAR_TOP': '3388933b59444c5db71fade0bbfef470',
   # 'CAM_FRONT': '020d7b4f858147558106c504f7f31bef',
   # 'CAM_FRONT_RIGHT': '16d39ff22a8545b0a4ee3236a0fe1c20',
   # 'CAM_BACK_RIGHT': 'ec7096278e484c9ebe6894a2ad5682e9',
   # 'CAM_BACK': 'aab35aeccbda42de82b2ff5c278a0d48',
   # 'CAM_BACK_LEFT': '86e6806d626b4711a6d0f5015b090116',
   # 'CAM_FRONT_LEFT': '24332e9c554a406f880430f17771b608'},
   # 'anns': ['173a50411564442ab195e132472fde71',
   # '5123ed5e450948ac8dc381772f2ae29a',
   # 'acce0b7220754600b700257a1de1573d',
   # '8d7cb5e96cae48c39ef4f9f75182013a',
   # 'f64bfd3d4ddf46d7a366624605cb7e91',
   # 'f9dba7f32ed34ee8adc92096af767868',
   # '086e3f37a44e459987cde7a3ca273b5b',
   # '3964235c58a745df8589b6a626c29985',
   # '31a96b9503204a8688da75abcd4b56b2',
   # 'b0284e14d17a444a8d0071bd1f03a0a2']}

   # explanation:
   # 'token' -- Code for searching and calling this sample
   # 'timestamp' -- the timestamp of this frame
   # 'prev'/'next' -- the frame of previous/next frame(before/after 0.1 sec).
   # `scene` -- the token of scene, which this sample belongs to.
   # `data` -- the token of data, which different sensors collect at this time frame, we could see different radar, lidar and cameras
   # `anns` -- the token of annotated objects in this frame
   ```

3. sample_data

   sample data means the data collected by a specific sensor in a frame. In part of sample, we could know the dataset contains data collected by RADAR, LiDAR, and Cameras. Let us take look how lidar data looks like:

   ```python
   sensor = 'LIDAR_TOP'
   lidar_data = nusc.get('sample_data', my_sample['data'][sensor])
   lidar_data

   # output:
   # {'token': '3388933b59444c5db71fade0bbfef470',
   # 'sample_token': 'e93e98b63d3b40209056d129dc53ceee',
   # 'ego_pose_token': '3388933b59444c5db71fade0bbfef470',
   # 'calibrated_sensor_token': '7a0cd258d096410eb68251b4b87febf5',
   # 'timestamp': 1531883530449377,
   # 'fileformat': 'pcd',
   # 'is_key_frame': True,
   # 'height': 0,
   # 'width': 0,
   # 'filename': 'samples/LIDAR_TOP/n015-2018-07-18-11-07-57+0800__LIDAR_TOP__1531883530449377.pcd.bin',
   # 'prev': '',
   # 'next': 'bc2cd87d110747cd9849e2b8578b7877',
   # 'sensor_modality': 'lidar',
   # 'channel': 'LIDAR_TOP'}

   # explanation:
   # a set of 'token' -- code for corresponding object
   # 'timestamp' -- the timestamp of collection
   # 'fileformat' -- here the lidar data is in point cloud format
   # 'is_key_frame' -- keyframe means the frame is annotated
   # 'heigt'/'width' -- 3D lidar data does not have these attributes
   # 'filename' -- path of the stored data
   # 'prev'/'next' -- token of data in previous/next frame

   ```

   For visualization of the lidar data:

   ```python
   nusc.render_sample_data(lidar_data['token'])
   ```

   ![](https://raw.githubusercontent.com/adskenlin/adskenlin.github.io/master/2021/08/16/Deep-Learning-on-NuScenes-LiDAR-Data/lidar_vis.png)

4. sample_annotation

   sample_annotation refers to the anntated information of an object in a sample.

   ```python
   my_annotation_token = my_sample['anns'][5]
   my_annotation_metadata =  nusc.get('sample_annotation', my_annotation_token)
   my_annotation_metadata

   # output:
   # {'token': 'f9dba7f32ed34ee8adc92096af767868',
   # 'sample_token': 'e93e98b63d3b40209056d129dc53ceee',
   # 'instance_token': '076e76a589dd4f40adce27b3f3377f58',
   # 'visibility_token': '4',
   # 'attribute_tokens': ['cb5118da1ab342aa947717dc53544259'],
   # 'translation': [1009.009, 598.528, 0.664],
   # 'size': [1.871, 4.478, 1.456],
   # 'rotation': [0.8844059033120215, 0.0, 0.0, 0.46671854279302766],
   # 'prev': '',
   # 'next': '21ece7170dfa431bb504e15f68fc40ce',
   # 'num_lidar_pts': 151,
   # 'num_radar_pts': 3,
   # 'category_name': 'vehicle.car'}

   # explanation:
   # a set of token -- code for corresponding object
   # 'translation' -- the global coordinate system of center of this object
   # 'size' -- the size(width, length, height) of this object
   # 'rotation' -- the orientation of the object shown in quaternion
   # 'num_lidar_pts' -- number of lidar points on this object
   # 'category_name' -- object class name 
   ```

   For visulization of this object:

   ```python
   nusc.render_annotation(my_annotation_token)
   ```

   ![](https://raw.githubusercontent.com/adskenlin/adskenlin.github.io/master/2021/08/16/Deep-Learning-on-NuScenes-LiDAR-Data/anns_vis.png)

## Data Format and Usage

As above shown, we've introduced different data formats in NuScenes dataset. The data formats and their corresponding functions could help us build data processing pipeline in future steps.

Before we build the data pre-processing pipeline, we need to think about: What kind of LiDAR data do we need for deep learning? An example of senor data processing from SOTA methods like Center-Net, MultiXNet shows the pipeline as follows: 

![](https://raw.githubusercontent.com/adskenlin/adskenlin.github.io/master/2021/08/16/Deep-Learning-on-NuScenes-LiDAR-Data/lidardata_pipeline.PNG)

We'll discuss data pre-processing methods and options in next part.

## Reference

[1] [nuScenes: A multimodal dataset for autonomous driving](https://arxiv.org/pdf/1903.11027.pdf)

[2] [Center-based 3D Object Detection and Tracking](https://arxiv.org/pdf/2006.11275.pdf)
