# EasyWDA

# CODE WILL BE UPLOADED WHEN THE PAPER IS RELEASED.

[Example video](./30.03.18_3_10_04.mp4)

EasyWDA is the first deep learning framework to detect and decode waggle dances in natural conditions.

This repository shows how to run our code on our data to obtain the same accuracy metrics as in our paper. You can also easily use it on your own data.

## 0-Dataset structure

Our dataset structure is as follows :
```
apisd_datasets
   |——————labels
            └——————Dancing
                     └——————30.03.18_1_1_00
                                └——————00001.txt (first frame of the video)
                                └——————00002.txt
                                └——————...
                     └——————30.03.18_1_1_01
                                └——————00048.txt (first frame of the dance)
                                └——————00049.txt
                                └——————...
                     └——————...
   └——————rgb-images
             └——————Dancing
                     └——————30.03.18_1_1_00
                                └——————00001.jpg
                                └——————00002.txt
                                └——————...
                     └——————30.03.18_1_1_01
                                └——————00001.jpg (first frame of the video)
                                └——————00002.jpg
                                └——————...
                     └——————...
   └——————splitfiles
            └——————trainlist01.txt (list of annotated frames included in training set)
            └——————vallist01.txt (list of annotated frames included in validation set)
            └——————testlist01.txt (list of annotated frames included in testing set)
            └——————finalAnnots.txt (file used for computing the video AP metric)
   └——————videos
            └——————30.03.18_1_1_00.mp4
            └——————30.03.18_1_1_01.mp4
            └——————...
   └——————trainlist.txt (list of folders of annotated frames included in training set)
   └——————vallist.txt (list of folders of annotated frames included in validation set)
   └——————testlist.txt (list of folders of annotated frames included in testing set)

```
You can prepare data using 'annotations.txt' and the video folder through the following command line:
`python dataset/make_dataset.py -videos`. 
Dataset files can be dowloaded at:

## 1-Dance Detection

For the detection of the dancing individuals, we adapted the original YOWOv2 network that can be found here : https://github.com/yjh0410/YOWOv2. This video-based approach is crucial to perform a detection of specific behaviors that would be impossible with a classic frame-based approach.

### Training (command line examples)

To retrain the network on your own dance data:
`python train.py --cuda -d apisd --root dataset/ --max_epoch 10 -v yowo_v2_nano --eval -K 16`

(You can also train models in a batch with multiple K values at the same time:
`./train_apisd_batch.sh`)

### Validation (command line examples)

To obtain frame mAP from the validation set:
`python eval.py --cuda -d apisd -v yowo_v2_nano -size 224 --weight weights/apisd/yowo_v2_nano/yowo_v2_nano_epoch_3.pth --cal_frame_mAP -K 32`

To obtain video mAP from the validation set:
`python eval.py --cuda -d apisd -v yowo_v2_nano -size 224 --weight weights/apisd/yowo_v2_nano/yowo_v2_nano_epoch_3.pth --cal_video_mAP -K 32`

(You can also run validation in a batch with multiple K values at the same time:
`./evalf_batch.sh`
`./evalv_batch.sh`)

You can get validation graphs:

## 2-Dance Tracking

For the tracking within dances, we used the deep-sort-realtime package : https://pypi.org/project/deep-sort-realtime/.

### Hyperparameters tuning using outputs from "1-Dance Detection" as an input and with the validation set (command line examples)

You first need to infer outputs from the detection part:
`python infer.py --cuda -d apisd -v yowo_v2_nano -size 224 --weight weights/apisd/nano_K16_default/yowo_v2_nano_epoch_10.pth --video dataset/apisd/videos/30.03.18_4_16_02.mp4 -K 32 --show
`

(You can also infer outputs for multiple videos at the same time:
`./infer_batch.sh`)

Then you track dances through videos:
`track.py`

Then you thieve dances by removing short length ones:
`thieve_tracking.py`

Then you apply pca to get rid of detections that are not part of the dances:
`remove_returnrun.py`

Then you prepare ...

### Validation (command line examples)

WIP

## 3-Decoding

Decoding consistes in the translation of detected dances to geolocalised coordinates.
