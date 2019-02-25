Training YOLO with BBox-Label-Tool 
==================================

A simple tool for labeling object bounding boxes in images, implemented with Python Tkinter.

## Dependencies

```
sudo apt-get install python-imaging-tk
```
## Clone BBox-Label-Tool

```
git clone https://github.com/Texas-Aerial-Robotics/BBox-Label-Tool.git
```

## Training instructions

The following are a modified version of the instructions found on the yolo website

*1.* Start by collecting images. the goal is to collect close to 500 img/class 

*2.* create a new folder in BBox-Label-Tool/Images  

*3.* For multi-class task, modify 'class.txt' with your own class-candidates and before labeling bbox,

ex
```
person
dog
airplane
``` 

*4.* run main.py and annotate images. The current tool requires that **the images to be labeled, reside in /Images/001, /Images/002, etc. You will need to modify the code if you want to label images elsewhere**. For multi-class choose the 'Current Class' in the Combobox and make sure you click 'ComfirmClass' button. 

*5.* To create a new bounding box, left-click to select the first vertex. Moving the mouse to draw a rectangle, and left-click again to select the second vertex.
  - To cancel the bounding box while drawing, just press `<Esc>`.
  - To delete a existing bounding box, select it from the listbox, and click `Delete`.
  - To delete all existing bounding boxes in the image, simply click `ClearAll`.

*6.* After finishing one image, click `Next` to advance. Likewise, click `Prev` to reverse. Or, input an image id and click `Go` to navigate to the speficied image.
  - Be sure to click `Next` after finishing a image, or the result won't be saved. 

*7.* We now need to convert the bbox labels into yolo readable form for this we will modify the convert script in `BBox-Label-Tool/Scripts` 

the `mypath` variable should path to the labels folder for your data set. Yolo will look for the annotations in the same folder as the image, so path the `outpath` variable to the image folder. the dataset is the name of the data set ex `002`. Then fill the classes array to match your `classes.txt`

```
mypath = "/home/eric/BBox-Label-Tool/Labels/002/" /
outpath = "/home/eric/BBox-Label-Tool/Images/002/"
dataSet = "002"
classes = ["person", "dog","airplane"]
```

*8.* run the convert.py script to format the annotations in yolo form.

*9.* combine your dataset_list files using your favorite text editor into train.txt and test.txt. It is recommended to not train on one of your data sets to use as a accuracy metric. 

*10.* Set up the darknet .data file 

- Create a file in darknet/cfg with the name <name-of-data>.data 
- Create a folder to store the output weights in. ex /home/eric/backup     
```
classes= 20
train  = <path-to-data>/train.txt
valid  = <path-to-data>/test.txt
names = data/<name-of-data>.names
backup = <path-to-backup-folder>
```

*11.* Make your <name-of-date>.names file in `darknet/data`. This file should be a coppy of the classes.txt file in `BBox-Label-Tool`. 

ex.
```
person
dog
airplane
``` 

*12.* Start your training run. You will fist need to select wich yolo model you would like to use the options are 

- yolov1: Not recommended. this model is old 
- yolov2: more accurate, and faster. 
- yolov3: about as fast as v2, but more accurate. Yolo v3 has a high GPU ram requirement to train and run. If your graphics card does not have enough ram, use yolo v2 
- tiny-yolo: Very fast yolo model. Would recommend for application where speed is most important. Works very well on Nvidia Jetson

The below command start training with yolo v3. Change the .cfg file to train with a different model
```
./darknet detector train cfg/<nameofdata>.data cfg/yolov3.cfg <weights(if you want to continue training from a backup)>
```

*13.* After training has completed you can test the output weights by using the following command. 
```
./darknet detector test cfg/<nameofdata>.data cfg/yolov3.cfg <weights> <test-picture>
```


**Screenshot:**
![Label Tool](./screenshot.png)

Data Organization
-----------------
LabelTool  
** |**   
**| --main.py   source code for the tool**  
** |**   
**| --Images/   **direcotry containing the images to be labeled** 
** |**   
** | --Labels/   **direcotry for the labeling results**  
** | ** 
**| --Labels-Yolo-Format **direcotry for the labeling results in yolo readable format**
**|**   
**|** --Examples/  **direcotry for the example bboxes**  

