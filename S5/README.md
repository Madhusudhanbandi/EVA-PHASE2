
**Human Pose Estimation**

**Data**

1.1 Pose Estimation on COCO The COCO train, validation, and test sets contain more than 200k images and 250k person instances labeled with keypoints. 150k instances of them are publicly available for training and validation. Our models are only trained on all COCO train2017 dataset (includes 57K images and 150K person instances) no extra data involved, ablation are studied on the val2017 set and finally we report the final results on test-dev2017 set to make a fair comparison with the public state-of-the-art results


1.2 Pose Estimation and Tracking on PoseTrack PoseTrack dataset is a large-scale benchmark for multi-person pose estimation and tracking in videos. It requires not only pose estimation in single frames, but also temporal tracking across frames. It contains 514 videos including 66,374 frames in total, split into 300, 50 and 208 videos for training, validation and test set respectively. For training videos, 30 frames from the center are annotated. For validation and test videos, besides 30 frames from the center, every fourth frame is also annotated for evaluating long range articulated tracking.



**Architecture**

There are 2 main parts of architecture

Pose Estimation

Pose Tracking

Pose Estimation(Npose)

To discuss simplicity of architecture 3 arctitectures are discussed. It seems that obtaining high resolution feature, maps is crucial, but no matter how.

A. HourGlass

It features in a multi-stage architecture with repeated bottom-up, top-down processing and skip layer feature concatenation. B. Cascaded Pyramid Network (CPN)
It also involves skip layer feature concatenation and an online hard keypoint mining step. Both works above, use upsampling to increase the feature map resolution and put convolutional parameters in other blocks.

C. Authors Network

This method combines the upsampling and convolutional parameters into deconvolutional layers in a much simpler way, without using skip layer connections. This method simply adds a few deconvolutional layers over the last convolution stage in the ResNet, called C5. It adopts this structure because it is arguably the simplest to generate heatmaps from deep and low resolution features and also adopted in the state-of-the-art Mask R-CNN.

Three deconvolutional layers with batch normalization and ReLU activation are used

Each layer has 256 filters with 4 × 4 kernel \ The stride is 2 \ A 1 × 1 convolutional layer is added at last to generate predicted heatmaps {H1, H2,... Hk} for all k key points \ Joint Mean Squared Error (MSE) is used as the loss between the predicted heatmaps and targeted heatmaps. The targeted heatmap Hk for joint k is generated by applying a 2D gaussian centered on the kth joint’s ground truth location.


**Joint MSE Loss**

It is nothing but extension of MSE between joints obtained in predicted heatmaps and targeted heatmaps.

Algorithm is as follows:

for each joint obtained in GroundTruth heatmap:
loss = loss + MSE(heat map on GT, heat map on pred)
Above algorithm can further be extended if we want to give different weightage to differnt joint based on priority,
loss = loss + MSE(jointweightheat map on GT, joint weightheat map on pred)

**Pose Tracking**

Multi-person pose tracking in videos first estimates human poses in frames, and then tracks these human pose by assigning a unique identification number (id) to them across frames. Old method: The winner of ICCV’17 PoseTrack Challenge solves this multi-person pose tracking problem by first estimating human pose in frames using Mask RCNN, and then performing online tracking using a greedy bipartite matching algorithm frame by frame. The greedy matching algorithm is to first assign the id of Pik−1 in frame Ik−1 to Pjk in frame Ik if the similarity between Pik−1 and Pjk is the highest, then remove these two instances from consideration, and repeat the id assigning process with the highest similarity. When an instance Pjk in frame Ik has no existing Pik−1 left to link, a new id number is assigned, which indicates a new instance comes up.

Authors method: 2 major differences from above \

We have two different kinds of human boxes, one is from a human detector(e.g. Faster-RCNN, R-FCN) and the other are boxes generated from previous frames using optical flow.
Simply applying a detector designed for single image level to videos could lead to missing detections and false detections due to motion blur and occlusion introduced by video frames. Author propose to generate boxes for the processing frame from nearby frames using temporal information expressed in optical flow.

We propose to use a flow-based pose similarity metric.
Using bounding box IoU(Intersection-over-Union) as the similarity metric (SBbox) to link instances could be problematic when an instance moves fast thus the boxes do not overlap.

A more fine-grained metric could be a pose similarity (SP ose) which calculates the body joints distance between two instances using Object Keypoint Similarity (OKS). The pose similarity could also be problematic when the pose of the same person is different across frames due to pose changing.

Considering consecutive two frames is not enough, thus we have the flow-based pose similarity considering multi frames, denoted as SMulti−flow, meaning the propagated J^k comes from multi previous frames.



