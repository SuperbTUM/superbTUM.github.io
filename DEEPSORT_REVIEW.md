---
title: "Literature Review of Deepsort, 2017"
permalink: /DEEPSORT_REVIEW/
layout: page
theme: jekyll-theme-cayman
---

# Re-identification problem in pedestrian tracking with Siamese Network and semi-supervised learning



The major challenge of Re-ID problem is to deal with hard sample pairs (front paired with end) and achieve real-time tracking (≥30 FPS). Many SOTA works leverage GAN variants to deal with hard matching and advanced training strategy to achieve real-time tracking. To simplify our work, we won't use fancy tricks because these tricks are so tricky that we may get worse result. We will use the classical tracking method -- [DeepSort 2017](https://arxiv.org/abs/1703.07402), as our baseline. We hope to address the aforementioned two problems, or some what, strike a balance.

DeepSort is built based on Sort, with deep reID model as feature extractor for matching. The core idea of matching is to apply Siamese network to two candidates and see the distance between these two. In the final implementation of DeepSort, the distance used is cosine similarity (not sure why not using Mahalanobis distance at last). If the distance exceeds a certain limit (in DeepSort, the threshold is 0.2), the two candidates will not be matched.

In the naive version of DeepSort, it came up with a residual style network in only 2M parameters as reID model. This is replaced by many implementers with fast-reID model toolbox by JD.com, where you can get access in [Github](https://github.com/JDAI-CV/fast-reid). The loss function of the re-ID model varies, from CE loss to triplet loss or circle loss by Megvii. There is also [a paper](https://arxiv.org/pdf/1812.00442.pdf) introducing cosine softmax classifier with CE loss. The naive implementation of DeepSort uses CE loss. The model was trained with Market1501 and Mars. In this case, we can firstly switch to different Re-ID models and loss functions. We also leverage some augmentations like Cutout.

DeepSort leverages kalman filter as tracker. Tracker for MOT is a critical component. The kalman filter is a non-memorial tracker, and only takes previous state into consideration (predict) and rectifies itself in current state (update). Moreover, kalman filter assumes all objects are moving in a constant velocity. This is a naive assumption and may not be fit to various scenarios. To fully leverage the past tracking information, we intend to use a sequential model as tracker instead, for instance, RNN. We keep it very light with shared weight and a fixed context window, whose size could be empirical, based on statistics in continuous frames in benchmark data, to adjust to flexible tracklets. This means we need a temporary storage of all available tracklets within a time window. If the tracklet is shorter, we could have the dedicated output from a certain spot as the next state prediction. We have the last output as our final output for prediction of next location. The RNN model could be understood in the following way: each output of time sequence i represents the next state prediction (time i+1). The loss function of the RNN model is DIOU loss. The way of calculating loss is the mean of pairwise loss of each prediction and ground truth. 

What if we don't want to use GAN but want to tackle with hard pairs? (It's slow?) We can define two Siamese networks, one for front-front pair or back-back pair and the other one for front-back pair. And we also need a simple binary classifier (may be I can try multi-layer perceptron with Sigmoid) to determine the detection is front to the camera or end to the camera. We can design a customized Cutout algorithm to do augmentation. We have two dedicated areas to mask, one is facial area, the other is non-facial area. For a front image, if we mask the faces, we should get the `back` result, otherwise we should get the `front` result. The augmentation relies on pretrained facial detector. The classifier also needs to be pretrained with some classical re-id datasets. We could do clustering first to form fake labels. With the binary classifier, we can split the matching task to a certain Siamese network.

Others will remain immutable.

Rules of running the code: training dataset should have the same number of classes as testing dataset! The resolution of the video is 128*128. (If you want to change the resolution of the video, you need to modify the transformation part as well as the backbone average pooling part.)



* Dataset: [MOT16 Challenge](https://motchallenge.net/data/MOT16/). The ground truth bounding box information for each detection is a 9-dim vector. The first number shows the number of frames the object shows up. The second number shows the trajectory the object belongs to and -1 means no trajectory was assigned. The following four numbers are coordinates of bounding box with format of (top, left, width, height). Following the number is the confidence score. The last two digits are reserved to -1 in the detection stage. In the matching stage, the 8th number refers to the object type (the classification is not binary in MOT16 however in our project, we focus only on pedestrians) and 9th number is the visibility ratio of the bounding box. Note that this could be affected by occlusion or image border cropping.

* Tools: `ffmpeg`

`ffmpeg` is a very classical tool for video processing and is compatible with various platforms.

Check the meta info of the video: `ffmpeg -i video_name`. Critical info include format and encoder.

Check the frame info of the video: `ffprobe -i video_name -print_format json -loglevel fatal -show_streams -count_frames -select_streams v` and returns json-format results. Critical info include duration and number of frames.

Uniformly extract frames from the video: `ffmpeg -i test.mp4 -vf fps=fps,showinfo -f image2 output_dir/%08d.jpg` where $fps=\frac{frames}{duration}$

* Evaluation: MOTA

MOTA is the abbreviation of multiple object tracking accuracy. $MOTA=1-\frac{\sum FN_t+FP_t+IDSW_t}{\sum GT_t}$

