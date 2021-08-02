# MMDetection 学习笔记

## 1. 使用已有模型和标准数据集进行推断和训练

### 1.1 用已有模型进行推断

在MMDetection中，一个模型由一个配置文件和存储在`checkpoint file` 中的文件定义。

作为开始，我们推荐使用Faster RCNN 的配置文件和对应的checkpoint file.推荐将checkpoint 文件下载到`checkpoint`文件夹中。 


~~~python
from mmdet.apis import init_detector, inference_detector
import mmcv

# 指定模型配置文件和checkpoint文件的路径
config_file = ''
checkpoint_file = ''

# 从config file 和 checkpoint file生成模型
model = init_detector(config_file, checkpoint_file,device = "cuda:0")

# test a single image and show the results
img = "test.jpg"
result = inference_detector(model, img)

# visualize the results in a new window
show_result_pyplot(model,img,result)

# or save the visualization results to image files
show_result_pyplot(img, result, out_file='result.jpg')

# test a video and show the results
video = mmcv.VideroReader('video.mp4')
for frame in video:
    result = inference_detector(model, frame)
    show_result_pyplot(model, frame, result, wait_time = 1)
~~~



### 1.2 异步接口

通过使用CUDA 流，允许在GPU的接口代码中不阻塞CPU，并且对单线程的应用，可以更好的利用CPU/GPU。推理可以在不同的输入数据样本或者推理流程的不同模型之间同时进行。

使用`async_benchmark.py`来比较同步和异步接口的速度

~~~python
import asyncio
import torch
from mmdet.apis import init_detector, async_inference_detector
from mmdet.utils.contextmanagers import concurrent

async def main():
    config_file = ""
    checkpoint_file = ""
    device = "cuda:0"
    model = init_detector(config_file, checkpoint_file, device)
    
    #queue is used for concurrent inference of multiple images
    streamqueue = aysncio.Queue()
    #queue_size defines concurrency level
    streamqueue_size = 3
    for _ in range(streamqueue_size):
        streamqueue.put_nowait(torch.cuda.Stream(device = device))
     
    # test a single image and show the results
    img = "test.jpg"
    
    async with concurrent(streamqueue):
        result  = await async_inference_detector(model, img)
        
    
    # visualize the results in a new window
	model.show_result(img,result)
  
asyncio.run(main())
~~~

### 1.3 Demos

我们提供了三个demo脚本，由高级API实现。

+ Image demo

~~~python
python demo/image_demo.py \
${IMAGE_FILE}\
${CONFIG_FILE}\
${CHECKPOINT_FILE}\
[--device ${GPU_ID}]\
[--score-thr ${SCORE_THRE}]
~~~



+ Webcam demo

~~~python
python demo/webcam_demo.py \
${CONFIG_FILE}\
${CHECKPOINT_FILE}\
[--device ${GPU_ID}] \
[--camera-id ${CAMERA-ID}] \
[--score-thr ${SCORE_THRE}]
~~~



~~~python
python demo/webcam_demo.py \
configs/faster_rcnn/faster_rcnn_r50_fpn_1x_coco.py \
checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pt
~~~



+ Video demo

~~~python
python demo/video_demo.py \
${VIDEO_FILE} \
${CONFIG_FILE} \
${CHECKPOINT_FILE} \
[--device ${GPU_ID}] \
[--out ${OUT_FILE}] \
[--show] \
[--wait-time ${WAIT_TIME}]
~~~

~~~python
python demo/video_demo.py demo/demo.mp4 \
    configs/faster_rcnn/faster_rcnn_r50_fpn_1x_coco.py \
    checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth \
    --out result.mp
~~~



### 1.4 在标准数据集上测试模型

为了评测一个模型的精确度，通常在标准数据集上测试模型，MMDetection支持多个公开的数据集包括COCO，Pascal VOC,CItyScapes等，这个小节将会展示怎么在支持的数据集上测试已存在的模型。

#### 1.4.1 准备数据集

公开数据集像Pascal VOC 或者 COCO都可以在官方网站获得。

Notes:在检测的任务中，Pascal COV2012 是Pascal VOC2007的拓展（没有重叠），我们可以一起使用它们。推荐方式在工程文件夹之外的某个地方下载并解压数据集，并且使用符号链接将数据集的根目录连接到`$MMDetection/data`下。如果你的文件夹结构不同，你可能需要在配置文件中改变对应的路径。

```
mmdetection
├── mmdet
├── tools
├── configs
├── data
│   ├── coco
│   │   ├── annotations
│   │   ├── train2017
│   │   ├── val2017
│   │   ├── test2017
│   ├── cityscapes
│   │   ├── annotations
│   │   ├── leftImg8bit
│   │   │   ├── train
│   │   │   ├── val
│   │   ├── gtFine
│   │   │   ├── train
│   │   │   ├── val
│   ├── VOCdevkit
│   │   ├── VOC2007
│   │   ├── VOC2012
```

一些模型要求额外的COCO-stuff 数据集，像HTC, DetectorRS 和 SCNet,你可以下载和解压然后把它们移到coco文件夹。文件夹看起来应该是下面这样:

```
mmdetection
├── data
│   ├── coco
│   │   ├── annotations
│   │   ├── train2017
│   │   ├── val2017
│   │   ├── test2017
│   │   ├── stuffthingmaps
```

cityscapes标注需要使用`tools/dataset_converters/cityscapes.py`转换成coco格式:

```
pip install cityscapesscripts

python tools/dataset_converters/cityscapes.py \
    ./data/cityscapes \
    --nproc 8 \
    --out-dir ./data/cityscapes/annotations
```

#### 1.4.2 测试已存在的模型

我们提供了测试脚本来在整个数据集上(COCO,PASCAL VOC,Cityscapes,etc...)测试模型。支持下面的测试环境

+ single GPU
+ single node multiple GPUS
+ multiple nodes

根据环境选择合适的脚本来进行测试:

```
# single-gpu testing
python tools/test.py \
    ${CONFIG_FILE} \
    ${CHECKPOINT_FILE} \
    [--out ${RESULT_FILE}] \
    [--eval ${EVAL_METRICS}] \
    [--show]

# multi-gpu testing
bash tools/dist_test.sh \
    ${CONFIG_FILE} \
    ${CHECKPOINT_FILE} \
    ${GPU_NUM} \
    [--out ${RESULT_FILE}] \
    [--eval ${EVAL_METRICS}]
```

`tools/dist_test.sh`也支持多节点测试，但是依赖于PyTorch的`launch utility`

**例子**

假设你已经下载了checkpoints 到`checkpoints`文件夹

1. 测试Faster RCNN并且可视化结果。按下任意键测试下一个图像。

   ```
   python tools/test.py \
       configs/faster_rcnn/faster_rcnn_r50_fpn_1x_coco.py\   checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth \
       --show
   ```

2. 测试Faster RCNN并且保存绘制的图像给未来的可视化。

~~~python
python tools/test.py \
    configs/faster_rcnn/faster_rcnn_r50_fpn_1x.py \
    checkpoints/faster_rcnn_r50_fpn_1x_coco_20200130-047c8118.pth \
    --show-dir faster_rcnn_r50_fpn_1x_results
~~~

`--show-dir`:指定存储结果的路径

3. 在PASCAL VOC(没有保存测试结果)上测试FasterRCNN并且评估mAP.

~~~python
python tools/test.py \
    configs/pascal_voc/faster_rcnn_r50_fpn_1x_voc.py \
    checkpoints/faster_rcnn_r50_fpn_1x_voc0712_20200624-c9895d40.pth \
    --eval mAP
~~~

`--eval-options` :指定键值对选项，eval配置将会是dataset.evaluate()的关键字，它只是用来评估的。

4. 使用8张显卡来测试Mask RCNN,并且评估bbox和mask AP.

~~~python
./tools/dist_test.sh \
    configs/mask_rcnn_r50_fpn_1x_coco.py \
    checkpoints/mask_rcnn_r50_fpn_1x_coco_20200205-d4b0c5d6.pth \
    8 \
    --out results.pkl \
    --eval bbox segm
~~~

