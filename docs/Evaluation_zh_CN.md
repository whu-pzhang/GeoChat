# 评测

We evaluate GeoChat on a variety of tasks, including scene classification, region captioning, visual grounding, grounding description and VQA.
Converted files in the input format for GeoChat are available at [GeoChat-Bench](https://huggingface.co/datasets/MBZUAI/GeoChat-Bench/tree/main)

## Results

1. Scene classification

执行以下命令：

```Shell
python geochat/eval/batch_geochat_scene.py \
    --model-path /data2/hf_models/geochat_7b \
    --image-folder data/GeoChat-Bench/images/UCMerced_LandUse/Images \
    --question-file data/GeoChat-Bench/UCmerced.jsonl \
    --answers-file outputs/UCmerced_geochat_7b.jsonl # output file
```


| Dataset  | Accuary |
| -------- | ------- |
| UCmerced | 84.48   |
| AID      |         |


2. VQA

GeoChat-Bench 数据集中提供的 RSVQA jsonl 文件，都没有提供问题的真值，导致没法直接计算指标。
因此，需要根据文章中对所用数据集的描述，从原始 RSVQA 数据集中重新提供包含真值信息的，用于评测的 `jsonl` 文件。


GeoChat-Bench 数据集采用的是 [RSVQA](https://rsvqa.sylvainlobry.com/#downloads) 数据集，详情参考论文 [RSVQA: Visual Question Answering for
Remote Sensing Data](https://ieeexplore.ieee.org/abstract/document/9088993)

以下为论文中对数据集的介绍：
>1. Low Resolution (LR): This data set is based on Sentinel2 images acquired over The Netherlands. Sentinel-2 satellites
>provide 10-m resolution (for the visible bands used in this
>data set) images with frequent updates (around five days) on a
>global scale. These images are openly available through ESA’s
>Copernicus Open Access Hub.
>
>2. High-Resolution (HR): This data set uses 15-cm resolution aerial RGB images extracted from the high-resolution
>orthoimagery (HRO) data collection of the USGS. This collection covers most urban areas of the USA, along with a
>few areas of interest (e.g., national parks). For most areas
>covered by the data set, only one tile is available with
>acquisition dates ranging from the year 2000 to 2016, with
>various sensors. The tiles are openly accessible through USGS’
>EarthExplorer tool.


- 数据集下载：分别下载 [Low resolution dataset](https://zenodo.org/records/6344334) 和 [High resolution dataset](https://zenodo.org/records/6344367)，按以下结构组织：

```bash
RSVQA
├── HR
│   ├── Images.tar
│   ├── USGS_split_test_phili_answers.json
│   ├── USGS_split_test_phili_images.json
│   ├── USGS_split_test_phili_questions.json
└── LR
    ├── all_answers.json
    ├── all_questions.json
    ├── Images_LR
    ├── Images_LR.zip
    ├── LR_split_test_answers.json
    ├── LR_split_test_images.json
    ├── LR_split_test_questions.json
    ├── LR_split_train_answers.json
    ├── LR_split_train_images.json
    ├── LR_split_train_questions.json
    ├── LR_split_val_answers.json
    ├── LR_split_val_images.json
    └── LR_split_val_questions.json
```

- 数据解压

```bash
cd HR
# 只需要解压 tif 格式文件即可
tar -xvf Images.tar --wildcards "Data/*.tif"  # 10659
cd ../LR
unzip Images_LR.zip
```

GeoChat 选用的是 RSVQA-HR 中的 test set 2 (covers the city of Philadelphia) 作为评测数据集，包含 47K 问答对（包含presence, comparison 和 rural/urban，忽略了 area 和 count 类别的问答）。对应原 RSVQA-HR 数据集中的文件如下：

- `USGS_split_test_phili_answers.json`
- `USGS_split_test_phili_images.json`
- `USGS_split_test_phili_questions.json`

>We split the data in a
>training set (61.5% of the tiles), a validation set (11.2%), and
>test sets (20.5% for test set 1, and 6.8% for test set 2). As it
>can be seen in Fig. 4, test set 1 covers similar regions as the
>training and validation sets, while the test set 2 covers the city
>of Philadelphia, which is not seen during the training. Note
>that this second test set also uses another sensor (marked as
>unknown on the USGS data catalog), not seen during training.

- RSVQA-LR 评测



```bash
python geochat/eval/batch_geochat_vqa.py \
    --model-path /data2/hf_models/geochat_7b \
    --image-folder data/GeoChat-Bench/images/Images_LR/ \
    --question-file data/GeoChat-Bench/lrben.jsonl \
    --answers-file outputs/lrben_geochat_7b.jsonl # output file
```


- RSVQA-HR 评测

```Shell
python geochat/eval/batch_geochat_vqa.py \
    --model-path /data2/hf_models/geochat_7b \
    --image-folder data/GeoChat-Bench/images/Images_LR/ \
    --question-file data/GeoChat-Bench/lrben.jsonl \
    --answers-file outputs/lrben_geochat_7b.jsonl # output file
```

| Dataset | Accuary |
| ------- | ------- |
| LRBEN   |         |
| HRBEN   |         |


1. Region-Captioning/Visual grounding




## Detials


Below we provide a general guideline for evaluating datasets.

1. LRBEN/HRBEN.
Images and ground truth for evaluation need to be downloaded from the following sources: [LRBEN](https://zenodo.org/records/6344334), [HRBEN](https://zenodo.org/records/6344367)
Give the path to the extracted image folder in the evaluation script. We add the following text after each question during our evaluation.
```
<question>
Answer the question using a single word or phrase.
```
```Shell
python geochat/eval/batch_geochat_scene.py \
    --model-path /data2/hf_models/geochat_7b \
    --image-folder data/GeoChat-Bench/images/UCMerced_LandUse/Images \
    --question-file data/GeoChat-Bench/UCmerced.jsonl \
    --answers-file outputs/UCmerced_geochat_7b.jsonl # output file
```
2. Scene Classification.
Download the images from the following sources, [UCmerced](http://weegee.vision.ucmerced.edu/datasets/landuse.html), [AID](https://drive.google.com/drive/folders/1-1D9DrYYWMGuuxx-qcvIIOV1oUkAVf-M). We add the following text after each question during our evaluation.
```
<question>
Classify the image from the following classes. Answer in one word or a short phrase.
```
```Shell
python geochat/eval/batch_geochat_scene.py \
    --model-path /path/to/model \
    --question-file path/to/jsonl/file \
    --answer-file path/to/output/jsonl/file \
    --image_folder path/to/image/folder/
```

3. Region-Captioning/Visual grounding.

The evaluation images are present in the image.zip folder in [GeoChat_Instruct](https://huggingface.co/datasets/MBZUAI/GeoChat_Instruct/blob/main/images.zip). 
```Shell
python geochat/eval/batch_geochat_grounding.py \
    --model-path /path/to/model \
    --question-file path/to/jsonl/file \
    --answer-file path/to/output/jsonl/file \
    --image_folder path/to/image/folder/
```

```Shell
python geochat/eval/batch_geochat_referring.py \
    --model-path /path/to/model \
    --question-file path/to/jsonl/file \
    --answer-file path/to/output/jsonl/file \
    --image_folder path/to/image/folder/
```