---
title: 'MMDetection3D: Deep Learning Toolbox for 3D LiDAR Data - P1'
date: 2021-09-07 21:09:30
tags:
- Deep Learning
- LiDAR
- Point Cloud Data
---
MMDetection3D is an open-source object detection codebase tool based on PyTorch. It's a project developed by MMLab, which has published several famous toolboxes e.g. MMDetection, MMsegmentation and so on. 

For more information, please visit their [official github repo](https://github.com/open-mmlab/mmdetection3d). In this blog, I would like to introduce this toolbox and explain how it works from its source code.

## Introduction

This toolbox was built focusing on 3D object detection tasks. If you've heard about `mmdetection` of Openmmlab, you could think this repo as an 3D-extension of basic `mmdetection`. It is built based on `mmdetection` and `mmcv`, and it use nearly same mechanism and coding-style as `mmdetection`.

I would highly recommend this toolbox for 3D Object Detection, since it has everything we need in this field and it works much more efficient than your own hand-written codes. Reasons:

+ It provides functions and built-in dataset classes to process raw 3D Data no matter it's from NuScenes, Lyft or Waymo.

+ It provides a huge number of functions from SOTA methods for processing point cloud data, anchors, bounding boxes, which we may use during our pre-processing step.

+ It contains various operations and their functions, which we may use in different parts of network, such as spare 3D convolution, iou3D calculation.

+ while designing architecture, it provides us various sub-architectures for ablation study. e.g. in different parts of network, such as in `backbone`, `Neck`, `Head`, it provides the architectures from many SOTA methods.

## Installation

Follow the steps [here](https://github.com/open-mmlab/mmdetection3d/blob/master/docs/getting_started.md). One thing you need to pay attention is, that the version of mmcv, python, pytorch and mmdetection should match. I was stucking in a problem for a long time, which causes the last installtion step `pip install -v -e .` failed. Because I have multiple CUDA versions(e.g 10.1, 11.2) installed and it does not match the pytorch version I used.

## Look into Codes

You could see many folders in `/mmdetection3d`. I would like sort them in 3 levels:

+ Sure to use: `configs`, `mmdet3d`,  `tools`

+ May use: `data`, `tests`

+ not relevant: `others`

### What is Configs?

In `configs`, you will have to create the config file for your deep learning methods, which should include the information of dataset setup, neural network architecture, training and validation parameters optimization and so on.

The basic mechanism of this toolbox is, that it has set up all the definitions, parameters in its registry module. When you want to use one of them, you only have to write it in config file and it will be called up automatically. 

To explain it clearly, let's look into an example, Centerpoint: 

```python

# read basic config files of different parts
# for dataset, use nus-3d.py as default
# for network architecture, use centerpoint_01voxel_second_secfpn_nus.py as default
# for training params and runtime setup, use cyclic_20e.py and default_runtime.py as default
_base_ = [
    '../_base_/datasets/nus-3d.py',
    '../_base_/models/centerpoint_01voxel_second_secfpn_nus.py',
    '../_base_/schedules/cyclic_20e.py', '../_base_/default_runtime.py'
]

# In the following part you have to set up the parameters
# which should overwrite the default values

# in this part,
# point_cloud_range and class_names are common params
# they may be used not only in dataset, 
# and also in models and training loops
point_cloud_range = [-51.2, -51.2, -5.0, 51.2, 51.2, 3.0]
class_names = [
    'car', 'truck', 'construction_vehicle', 'bus', 'trailer', 'barrier',
    'motorcycle', 'bicycle', 'pedestrian', 'traffic_cone'
]


# in this part,
# the params for setting up a model will be partially overwritten
model = dict(
    pts_voxel_layer=dict(point_cloud_range=point_cloud_range),
    pts_bbox_head=dict(bbox_coder=dict(pc_range=point_cloud_range[:2])),
    train_cfg=dict(pts=dict(point_cloud_range=point_cloud_range)),
    test_cfg=dict(pts=dict(pc_range=point_cloud_range[:2])))


# in this part,
# there are definitions about dataset, data path,
# database sampler, preprocessing pipeline for training and testing,
# they are necessary for setting up a dataset object
dataset_type = 'NuScenesDataset'
data_root = 'data/nuscenes/'
file_client_args = dict(backend='disk')

db_sampler = dict(
    data_root=data_root,
    info_path=data_root + 'nuscenes_dbinfos_train.pkl',
    rate=1.0,
    prepare=dict(
        filter_by_difficulty=[-1],
        filter_by_min_points=dict(
            car=5,
            truck=5,
            bus=5,
            trailer=5,
            construction_vehicle=5,
            traffic_cone=5,
            barrier=5,
            motorcycle=5,
            bicycle=5,
            pedestrian=5)),
    classes=class_names,
    sample_groups=dict(
        car=2,
        truck=3,
        construction_vehicle=7,
        bus=4,
        trailer=6,
        barrier=2,
        motorcycle=6,
        bicycle=6,
        pedestrian=2,
        traffic_cone=2),
    points_loader=dict(
        type='LoadPointsFromFile',
        coord_type='LIDAR',
        load_dim=5,
        use_dim=[0, 1, 2, 3, 4],
        file_client_args=file_client_args))

train_pipeline = [
    dict(
        type='LoadPointsFromFile',
        coord_type='LIDAR',
        load_dim=5,
        use_dim=5,
        file_client_args=file_client_args),
    dict(
        type='LoadPointsFromMultiSweeps',
        sweeps_num=9,
        use_dim=[0, 1, 2, 3, 4],
        file_client_args=file_client_args,
        pad_empty_sweeps=True,
        remove_close=True),
    dict(type='LoadAnnotations3D', with_bbox_3d=True, with_label_3d=True),
    dict(type='ObjectSample', db_sampler=db_sampler),
    dict(
        type='GlobalRotScaleTrans',
        rot_range=[-0.3925, 0.3925],
        scale_ratio_range=[0.95, 1.05],
        translation_std=[0, 0, 0]),
    dict(
        type='RandomFlip3D',
        sync_2d=False,
        flip_ratio_bev_horizontal=0.5,
        flip_ratio_bev_vertical=0.5),
    dict(type='PointsRangeFilter', point_cloud_range=point_cloud_range),
    dict(type='ObjectRangeFilter', point_cloud_range=point_cloud_range),
    dict(type='ObjectNameFilter', classes=class_names),
    dict(type='PointShuffle'),
    dict(type='DefaultFormatBundle3D', class_names=class_names),
    dict(type='Collect3D', keys=['points', 'gt_bboxes_3d', 'gt_labels_3d'])
]
test_pipeline = [
    dict(
        type='LoadPointsFromFile',
        coord_type='LIDAR',
        load_dim=5,
        use_dim=5,
        file_client_args=file_client_args),
    dict(
        type='LoadPointsFromMultiSweeps',
        sweeps_num=9,
        use_dim=[0, 1, 2, 3, 4],
        file_client_args=file_client_args,
        pad_empty_sweeps=True,
        remove_close=True),
    dict(
        type='MultiScaleFlipAug3D',
        img_scale=(1333, 800),
        pts_scale_ratio=1,
        flip=False,
        transforms=[
            dict(
                type='GlobalRotScaleTrans',
                rot_range=[0, 0],
                scale_ratio_range=[1., 1.],
                translation_std=[0, 0, 0]),
            dict(type='RandomFlip3D'),
            dict(
                type='PointsRangeFilter', point_cloud_range=point_cloud_range),
            dict(
                type='DefaultFormatBundle3D',
                class_names=class_names,
                with_label=False),
            dict(type='Collect3D', keys=['points'])
        ])
]

eval_pipeline = [
    dict(
        type='LoadPointsFromFile',
        coord_type='LIDAR',
        load_dim=5,
        use_dim=5,
        file_client_args=file_client_args),
    dict(
        type='LoadPointsFromMultiSweeps',
        sweeps_num=9,
        use_dim=[0, 1, 2, 3, 4],
        file_client_args=file_client_args,
        pad_empty_sweeps=True,
        remove_close=True),
    dict(
        type='DefaultFormatBundle3D',
        class_names=class_names,
        with_label=False),
    dict(type='Collect3D', keys=['points'])
]

data = dict(
    train=dict(
        type='CBGSDataset',
        dataset=dict(
            type=dataset_type,
            data_root=data_root,
            ann_file=data_root + 'nuscenes_infos_train.pkl',
            pipeline=train_pipeline,
            classes=class_names,
            test_mode=False,
            use_valid_flag=True,
            box_type_3d='LiDAR')),
    val=dict(pipeline=test_pipeline, classes=class_names),
    test=dict(pipeline=test_pipeline, classes=class_names))


# In this part,
# some params in runtime or training loop will be overwritten
evaluation = dict(interval=20, pipeline=eval_pipeline)
```

To draw a conclusion, the `configs` is where we define our network. We could use pre-defined definition or built-in functions to build up our neural network. If you think up a parameter or architecture which does not exist in toolbox, you could also registry them in next part `mmdet3d`.

### What is mmdet3d?

`mmdet3d` is the place where you could find and fetch your screwdriver. Under it, there are: `apis`, `core`, `dataset`, `models`, `ops`, `utils`.

+ In `apis` you could find the simple api-codes for training, testing and inference. 

+ In `core` you could find many pre-defined tools, which may be helpful in your research. E.g. `Anchor3DRangeGenerator` defines the method object to generate anchors. (for anchor-based methods, anchors could be generated in different ways)  

+ In `dataset` and `models` it provides various dataset/model options for your neural network. E.g. for models, you could choose ResNet as `backbone`, FPN as `neck`, anchor3dhead as `tast_heads`, and combine them as your network architecture.

+ In `ops` it provides many functions for data processing. E.g. Sparse 3D Convolution as an option of processing method for voxelized space.

If you want to design your own methods or architectures, you could just put the codes in corresponding folders and add your design to registry module. The toolbox could then fetch your self-designed object easily.

### What is tools?

In `tools`, it provides us codes for creating database, training, testing, parallel training with multi-GPUs or multi-workstations and so on. 

Briefly speaking, you'll set up all parameters and arguments of your neural network and training loop in `configs`, if you have some self-designed methods and architectures, you have to put and regitry them in `mmdet3d`. Afterwards you could run training and evaluation process with the codes in `tools`.

## What's coming NEXT

In my next blog 'MMDetection3D: Deep Learning Codebase for 3D LiDAR Data - P2', we will introduce the details of training in MMDetection3D.