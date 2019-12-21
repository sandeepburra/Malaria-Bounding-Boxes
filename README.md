# Malaria-Bounding-Boxes
## Introduction
Malaria is a disease caused by Plasmodium parasites that remains a major threat in global health, affecting 200 million people and causing 400,000 deaths a year.
For malaria as well as other microbial infections, manual inspection of thick and thin blood smears by trained microscopists remains the gold standard for parasite detection and stage determination because of its low reagent and instrument cost and high flexibility. Despite manual inspection being extremely low throughput and susceptible to human bias, automatic counting software remains largely unused because of the wide range of variations in bright field microscopy images.
## Machine Learning problem
The model should take raw image as input and gives output by drawing bounding box around cell and mention the type of cell. We can treat this problem as a Object Detection here objects are blood cells
## Data
I have used data from here, The data is comprised of microscopic images and annotations(Bounding Box information and class of each cell in the image).Data set consists of two classes of uninfected cells (RBCs and leukocytes) and four classes of infected cells (gametocytes, rings, trophozoites, and schizonts).Marked some cells as difficult if not clearly classified in one of the cell classes.
## Modeling with Faster RCNN
For training I have decided to use one of the popular state of the art object detection algorithm Faster R-CNN.
Faster R-CNN is composed of 3 neural networks - Feature Network, Region Proposal Network (RPN), Detection Network
### Feature Network:
The Feature Network is usually a well known pre-trained image classification network such as VGG minus a few last/top layers. The function of this network is to generate good features from the images. The output of this network maintains the the shape and structure of the original image ( i.e. still rectangular, pixels in the original image roughly gets mapped to corresponding feature "pixels", etc.)
### Region Proposal Network (RPN):
The RPN is usually a simple network with a 3 convolutional layers. There is one common layer which feeds into a two layers - one for classification and the other for bounding box regression. The purpose of RPN is to generate a number of bounding boxes called Region of Interests ( ROIs) that has high probability of containing any object. The output from this network is a number of bounding boxes identified by the pixel co-ordinates of two diagonal corners, and a value (1, 0, or -1, indicating whether an object is in the bounding box or not or the box can be ignored respectively ).
### Detection Network:
The Detection Network ( sometimes also called the RCNN network ) takes input from both the Feature Network and RPN , and generates the final class and bounding box. It is normally composed of 4 Fully Connected or Dense layers. There are 2 stacked common layers shared by a classification layer and a bounding box regression layer. To help it classify only the inside of the bounding boxes, the features are cropped according to the bounding boxes.

As our data is highly imbalanced, I did not get good results after training Faster R-CNN on whole data set. Whatever the cell is, our model is predicting it as Red Blood cell, as it is dominated class in our data set.To overcome this, I have divided this problem into two stage classification problem.
### First level classification:
* First I have created data set with only two lables, RBC and Other(all classes together except RBC)
* Faster R-CNN model was trained on this data, and this model predicts Bounding boxes and two class labels RBC and Other

### Second level classification:
* I have cropped the images with the ground truth bounding box dimensions of other class(Except RBC).
* Trained a dense net model on this cropped images,for all other classes.

## Testing 
* Raw image will be sent to first level classifier.
* It predicts classes RBC and OTHER with Bounding Boxes.This results(image_name, bounding box coordinates and class label) will be saved as a CSV file.
* In the second level, again take a raw image and crop other class cells with the predicted bounding box dimensions, finally feed to model 2. Model 2 will predict the original class label of the cell.
