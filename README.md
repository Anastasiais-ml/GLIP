# GDM-FLIP: Photographer Inspired Image Masking for Language-Image Model Pre-Training
![GDM-FLIP](./docs/GDM-FLIP.jpg)


# Abstract
We introduce a novel, straightforward, and effective technique for masking image patches in the training of the Vision-Language Model. Different from FLIP, which involves randomly masking and removing a significant portion of the image patches. This method employs a Gaussian Distribution to mask image patches (GDM-FLIP). This strategy is inspired by typical human photographic behavior, where subjects or objects are often centered within the image. GDM-FLIP incorporates this behavior by emphasizing the center of the image during training, enabling the model to efficiently learn the relationship between images and text without the need for complex computations. In comparative experiments on various pre-training datasets, GDM-FLIP outperformed FLIP across a range of downstream datasets and tasks. Furthermore, we conducted an analysis on different types of datasets to demonstrate GDM-FLIP’s advantages across diverse data contexts.

# Method

Random Masking            |  Gaussian Masking
:-------------------------:|:-------------------------:
![Random Masking](./docs/random_mask_image.png)  |  ![Gaussian Masking](./docs/gaussian_mask_image.png)

In contrast to the random mask strategy, our approach retains a greater number of image patches at the center of the image. 


# Results and Pre-trained Models

We pre-train the model according to the settings of [Open_clip](https://github.com/mlfoundations/open_clip)

**Zero-shot accuracy on ImageNet1K classification.**
We pre-trained the model for 30 epochs on the CC12M dataset by different image patch mask ratios with ViT-B/16 as the image encoder. Then we fine-tuned the FLIP and GDM-FLIP by an additional epoch.

| Method    | Masking | Inference Masking | Inference Unmasking | After Tuning |
|-----------|---------|-------------------|---------------------|--------------|
| CLIP      | -       | -                 | 35.5                | -            |
| FLIP      | 50%     | 32.1              | 34.0                | 34.2         |
| GDM-FLIP  | 50%     | 33.2              | **35.1**            | **35.4**     |
| FLIP      | 75%     | 26.3              | 29.4                | 30.0         |
| GDM-FLIP  | 75%     | 28.8              | 32.1                | 32.2         |
| FLIP      | 90%     | 16.6              | 21.1                | 22.0         |
| GDM-FLIP  | 90%     | 20.7              | 16.5                | 25.8         |


# Pre-training

Follow the instruction of [OpenCLIP](https://github.com/mlfoundations/open_clip) to pre-train the model with Patch Dropout.

Pre-training [FLIP](https://github.com/facebookresearch/flip/tree/main)

```bash
cd open_clip/src
torchrun --nproc_per_node=4 \
    -m training.main \
    --train-data '/data/cc12m/cc12m-train-{0000..2175}.tar' \
    --train-num-samples 10968539 \
    --dataset-type webdataset \
    --batch-size 320 \
    --force-patch-dropout 0.50 \
    --precision amp \
    --workers 4 \
    --imagenet-val /data/imagenet/validation/
```

Pre-training GDM-FLIP

```bash
cd open_clip/src
torchrun --nproc_per_node=4 \
    -m training.main \
    --train-data '/data/cc12m/cc12m-train-{0000..2175}.tar' \
    --train-num-samples 10968539 \
    --dataset-type webdataset \
    --batch-size 320 \
    --force-patch-dropout 0.50 \
    --normal-masking \
    --precision amp \
    --workers 4 \
    --imagenet-val /data/imagenet/validation/
```

# Unmasked tuning

```bash
cd open_clip/src
torchrun --nproc_per_node=4 \
    -m training.main \
    --train-data '/data/cc12m/cc12m-train-{0000..2175}.tar' \
    --train-num-samples 10968539 \
    --dataset-type webdataset \
    --pretrained /path/to/checkpoints/epoch_K.pt
    --batch-size 160 \
    --normal-masking \
    --precision amp \
    --workers 4 \
    --imagenet-val /data/imagenet/validation/
```

# Evaluation

We use [CLIP_benchmark](https://github.com/LAION-AI/CLIP_benchmark/tree/main) to evaluate CLIP, FLIP and GDM-FLIP on a standard set of datasets on different tasks.