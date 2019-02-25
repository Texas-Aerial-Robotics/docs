# YOLO Image Recognition 

## Detection Using A Pre-Trained Model

This post will guide you through detecting objects with the YOLO system using a pre-trained model. If you don't already have Darknet installed, you should do that first.

```
git clone https://github.com/pjreddie/darknet
cd darknet
make
```

Easy!

You already have the config file for YOLO in the cfg/ subdirectory. You will have to download the pre-trained weight file
```
wget https://pjreddie.com/media/files/yolov3.weights
```
Then run the detector!

```
./darknet detector test cfg/coco.data cfg/yolov3.cfg yolov3.weights data/dog.jpg
```
You will see some output like this:

```
layer     filters    size              input                output
    0 conv     32  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  32  0.299 BFLOPs
    1 conv     64  3 x 3 / 2   416 x 416 x  32   ->   208 x 208 x  64  1.595 BFLOPs
    .......
  105 conv    255  1 x 1 / 1    52 x  52 x 256   ->    52 x  52 x 255  0.353 BFLOPs
  106 detection
truth_thresh: Using default '1.000000'
Loading weights from yolov3.weights...Done!
data/dog.jpg: Predicted in 0.029329 seconds.
dog: 99%
truck: 93%
bicycle: 99%
```
Darknet prints out the objects it detected, its confidence, and how long it took to find them. We didn't compile Darknet with OpenCV so it can't display the detections directly. Instead, it saves them in predictions.png. You can open it to see the detected objects. Since we are using Darknet on the CPU it takes around 6-12 seconds per image. If we use the GPU version it would be much faster.

## Multiple Images

Instead of supplying an image on the command line, you can leave it blank to try multiple images in a row. Instead you will see a prompt when the config and weights are done loading:

```
./darknet detect cfg/yolov3.cfg yolov3.weights
layer     filters    size              input                output
    0 conv     32  3 x 3 / 1   416 x 416 x   3   ->   416 x 416 x  32  0.299 BFLOPs
    1 conv     64  3 x 3 / 2   416 x 416 x  32   ->   208 x 208 x  64  1.595 BFLOPs
    .......
  104 conv    256  3 x 3 / 1    52 x  52 x 128   ->    52 x  52 x 256  1.595 BFLOPs
  105 conv    255  1 x 1 / 1    52 x  52 x 256   ->    52 x  52 x 255  0.353 BFLOPs
  106 detection
Loading weights from yolov3.weights...Done!
Enter Image Path:
```
Enter an image path like data/horses.jpg to have it predict boxes for that image.

Once it is done it will prompt you for more paths to try different images. Use Ctrl-C to exit the program once you are done.

## Changing The Detection Threshold

By default, YOLO only displays objects detected with a confidence of .25 or higher. You can change this by passing the -thresh <val> flag to the yolo command. For example, to display all detection you can set the threshold to 0:

```
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg -thresh 0
```
Which produces:

![][all]

So that's obviously not super useful but you can set it to different values to control what gets thresholded by the model

## Tiny YOLOv3


We have a very small model as well for constrained environments, `yolov3-tiny`. To use this model, first download the weights:

```
wget https://pjreddie.com/media/files/yolov3-tiny.weights
```
Then run the detector with the tiny config file and weights:

```
./darknet detect cfg/yolov3-tiny.cfg yolov3-tiny.weights data/dog.jpg
```
## Compiling With CUDA and Opencv

Darknet on the CPU is fast but it's like 500 times faster on GPU! You'll have to have an Nvidia GPU and you'll have to install CUDA and opencv. If you have installed ROS desktop you should already have opencv installed 

## Install CUDA

https://docs.nvidia.com/cuda/cuda-installation-guide-linux/index.html

Once you have CUDA installed, change the first couple lines of the Makefile in the base directory to read:

```
GPU=1
OPENCV=1
```
Now you can make the project and CUDA will be enabled. By default it will run the network on the 0th graphics card in your system (if you installed CUDA correctly you can list your graphics cards using `nvidia-smi`). Again you can run 

```
./darknet detect cfg/yolov3.cfg yolov3.weights data/dog.jpg
```
## Note
Yolo v3 uses a lot of gpu ram. if you get the following error consider using tiny-yolo or yolov2 

grab pretrained weights for yolo V2 using 
```
wget https://pjreddie.com/media/files/yolov3.weights
```

run YOLOv2 

```
./darknet detect cfg/yolov2.cfg yolov2.weights data/dog.jpg
```
## Real-Time Detection on a Webcam

Running YOLO on test data isn't very interesting if you can't see the result. Instead of running it on a bunch of images let's run it on the input from a webcam!

To run this demo you will need to compile Darknet with CUDA and OpenCV. 

To run this demo you will need to compile Darknet with CUDA and OpenCV. Then run the command:
```
./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights
```
YOLO will display the current FPS and predicted classes as well as the image with bounding boxes drawn on top of it.

You will need a webcam connected to the computer that OpenCV can connect to or it won't work. If you have multiple webcams connected and want to select which one to use you can pass the flag -c <num> to pick (OpenCV uses webcam 0 by default).

You can also run it on a video file if OpenCV can read the video:
```
./darknet detector demo cfg/coco.data cfg/yolov3.cfg yolov3.weights <video file>
```



## Use YOLO to train your own custom classes 

@ref yoloTraining.md

## For more information on YOLO visit 

https://pjreddie.com/darknet/yolo/

## Cite

If you use YOLOv3 in your work please cite our paper!

@article{yolov3,
  title={YOLOv3: An Incremental Improvement},
  author={Redmon, Joseph and Farhadi, Ali},
  journal = {arXiv},
  year={2018}
}