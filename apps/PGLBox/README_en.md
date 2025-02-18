English | [中文](README.md)

## PGLBox: Distributed Hierarchical GPU Engine for Efficiently Training Super-large Scale Graph Neural Network


**PGLBox** is a super-large scale graph model training engine based on **GPU**. It breaks through the memory bottleneck by using heterogeneous hierarchical storage technology and supports graph sampling and training of billions of nodes and hundreds of billions of edges on a single machine. Users only need to perform simple configuration of the configuration file to use a single machine with multiple GPUs to train large-scale graph representation learning and quickly build GNN-based recommendation systems, user portraits, and graph retrieval systems.


<h4 align="center">
  <a href=#Quick-Start> Quick Start </a> |
  <a href=#Data-Format> Data Format </a> |
  <a href=#Features> Features </a> |
  Installation |
  Model Deployment
</h4>

## Update Log

- v2.1: Update some codes, including adding [designated training or infer nodes](./wiki/train_infer_from_file_ch.md) and adding [routine training function](./wiki/online_train.md). (2023.03.08)
- v2.0: PGLBox V2 version, supports multi-level storage of features and embeddings, and supports larger graph scale. (2022.12.29)
- v1.0: Add PGLBox capability, V1 version. (2022.12.14)


## Quick Start

In order to quickly use the capabilities of PGLBox, we provide some corresponding image environments. Just pull the image for the relevant hardware, download the corresponding data, modify the configuration file, and you can run it with one click. Currently, PGLBox only supports running on **v100** and **a100** GPUs.

```
docker pull registry.baidubce.com/paddlepaddle/pgl:pglbox-2.0-cuda11.0-cudnn8
```

After pulling the docker image, download the PGLBox code and enter the PGLBox directory.

```
git clone https://github.com/PaddlePaddle/PGL
cd PGL/apps/PGLBox
```

After entering the directory, just place some necessary graph data for our training graph representation model, such as node ID files and edge files. The data format will be introduced in detail later. Here we provide a large-scale academic citation network data MAG240M for quick start. Unzip the data file to the current directory.

```
wget https://baidu-pgl.gz.bcebos.com/pglbox/data/MAG240M/preprocessed_MAG240M.tar.gz
tar -zxf preprocessed_MAG240M.tar.gz
```

According to the graph structure information and the required model configuration, we can directly use [this configuration](./demo_configs/mag240m_metapath2vec.yaml) provided by us. The specific configuration meaning will be explained later.

Run the model in the PGLBox main directory using `nvidia-docker run` command:
```
nvidia-docker run -it --rm \
    --name pglbox_docker \
    --network host \
    --ipc=host \
    -v ${PWD}:/pglbox \
    -w /pglbox \
    registry.baidubce.com/paddlepaddle/pgl:pglbox-2.0rc-cuda11.0-cudnn8 \
    /bin/bash -c "/pglbox/scripts/train.sh ./demo_configs/mag240m_metapath2vec.yaml"
```

After training is complete, we can find the `mag240m_output` folder in the main directory. This folder contains `model` and `embedding` subfolders, which represent the saved model and node embeddings generated by inference, respectively.

## Data Format

For a detailed introduction to the graph data format and data preprocessing, please refer to [here](./wiki/data_format_ch.md).


## Features

#### <a href=#GPU-Accelerated-Experience> 🚀 GPU-Accelerated Experience </a>

#### <a href=#One-Click-Configurable-Complex-GNN-Model-Support>  📦 One-Click Configurable Complex GNN Model Support </a>

#### <a href=#Rich-Scenario-Solutions> 📖 Rich Scenario Solutions</a>


### GPU-Accelerated Experience

At the end of 2021, we open-sourced the [Graph4Rec](https://github.com/PaddlePaddle/PGL/tree/main/apps/Graph4Rec) toolkit, mainly used for large-scale graph node representation learning in recommendation scenarios. The toolkit is mainly used for large-scale training in multi-CPU scenarios and does not utilize the fast computing capabilities of GPUs. Therefore, we open-sourced the PGLBox GPU training framework this year, migrating the entire Graph4Rec process from CPU to GPU, greatly improving the overall training speed of the model. (Speed data TBD)

### One-Click Configurable Complex GNN Model Support

In industrial graph representation learning algorithms, in addition to the high demand for graph scale, there are also requirements for complex feature fusion, walking strategies, graph aggregation methods, diverse algorithm combinations, and routine training. We continue the design strategy of Graph4Rec, abstracting these real-world problems into several configuration modules to complete complex GNN support, adapting to **heterogeneous graph neural networks**, **meta-path random walk**, **large-scale sparse features** and other complex scenarios. We also provide model configuration files for different settings in the `./user_configs` directory for users to choose from.

<h2 align="center">
<img src="./../Graph4Rec/img/architecture.png" alt="graph4rec" width="800">
</h2>

In general, to complete a custom configuration, you need to complete **graph data preparation**, **graph walk configuration**, **GNN configuration**, and **training parameter configuration**. Due to different sample sizes and model computational complexities, the time and effect differences under different configurations are also significant. We provide a demonstration of the time consumption under different configurations on standard data (TBD) for reference.

<details><summary>Graph Data Preparation</summary>

Please refer to [here](./wiki/data_format_ch.md) for graph data preparation.

By default, PGLBox will train all nodes in the graph data and predict the embeddings of all nodes. If users only want to train a subset of nodes or predict a subset of nodes, PGLBox provides the corresponding function support. For details, please refer to [here](./wiki/train_infer_from_file_ch.md).

<br/>
</details>

<details><summary>Graph Walk Configuration</summary>
<br/>
The graph walk configuration is mainly used to control the specific parameters of the graph walk model. See below.

``` shell
# meta_path parameter, configure the walk path on the graph, here we take the MAG240M graph data as an example.
meta_path: "author2inst-inst2author;author2paper-paper2author;inst2author-author2paper-paper2author-author2inst;paper2paper-paper2author-author2paper"

# The window size of positive samples for the walk path
win_size: 3

# The number of negative samples corresponding to each pair of positive samples
neg_num: 5

# The depth of the meapath walk path
walk_len: 24

# Each starting node repeats walk_times walks, so that all neighbors of a node can be walked through as much as possible, making the training more uniform.
walk_times: 10
```

</details>

<details><summary>GNN Configuration</summary>
<br/>
The above graph walk configuration is mainly for metapath2vec model settings. On top of that, if we want to train more complex GNN graph networks, we can set the relevant configuration items of the GNN network for model adjustment.

``` shell
# GNN model switch
sage_mode: True

# Different GNN model choices, including LightGCN, GAT, GIN, etc. For details, please see the model folder of PGLBox.
sage_layer_type: "LightGCN"

# The proportion of node Embedding self-weight (sage_alpha) and the proportion of GNN aggregated node Embedding (1 - sage_alpha)
sage_alpha: 0.9

# The number of sampled node neighbors during graph model training
samples: [5]

# The number of sampled node neighbors during graph model inference
infer_samples: [100]

# GNN model activation layer selection
sage_act: "relu"
```

</details>

<details><summary>Model Training Parameter Configuration</summary>
<br/>
In addition to the above configurations, here are some relatively important configuration items.

``` shell
# Model type selection, currently default not to change. Later, we will provide more choices, such as ErnieSageModel, etc.
model_type: GNNModel

# Embedding dimension.
embed_size: 64

# Optimizer for sparse parameter server, currently supports adagrad and shared_adam.
sparse_type: adagrad

# Learning rate for sparse parameter server
sparse_lr: 0.05

# Loss function, currently supports hinge, sigmoid, nce.
loss_type: nce

# Whether to perform training. If you only want to separately warm start the model for inference, you can turn off need_train.
need_train: True

# Whether to perform inference. If you only want to train the model separately, you can turn off need_inference.
need_inference: True

# Number of training rounds
epochs: 1

# Batch size of training samples
batch_node_size: 80000

# Batch size of inference samples
infer_batch_size: 80000

# SSD cache trigger frequency
save_cache_frequency: 4

# Number of passes of dataset cached in memory
mem_cache_passid_num: 4

# Training mode, can be filled with WHOLE_HBM/MEM_EMBEDDING/SSD_EMBEDDING, default is MEM_EMBEDDING
train_storage_mode: MEM_EMBEDDING
  
```

</details>

In addition to the above configuration parameters, there are other configurations related to data, slot feature, model saving, and more. For more specific information, you can go to the provided `./user_configs` folder to view the specific yaml files, which have more detailed explanations for each configuration parameter.

### Providing Rich Scenario-based Solutions 

Below, we provide several scenario-based examples using **PGLBox**. Users can follow the scenario tutorials, replace data and configurations, and complete the corresponding model training and deployment.

- [Application on Graphs with No Attributes](./wiki/application_on_no_slot_features_ch.md)

- [Application on Graphs with Attributes](./wiki/application_on_slot_features_ch.md)

- [Application on Graphs with Edge Weights](./wiki/application_on_edge_weight_ch.md)

- [Application on Graphs with Multiple Edge Types](./wiki/application_on_multi_edge_types_ch.md)
