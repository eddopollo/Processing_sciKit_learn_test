# Phase-4 Milestone Project: Computer Vision Modeling for Pneumonia Diagnosis

Author: [Zeth Abney](https://www.linkedin.com/in/zeth-abney/)  
Pace: Flex  
Project Review data & time:   01/05/2023 11:00 AM    
Instructor Name: Morgan Jones  
Blog Post URL:  in-progress

# ![Overview](images/presentation_assets/Overview.png)     

The goal of this project is to develop a convolutional neural network (CNN) designed to view images of chest x-ray scans and classify the images as showing symptoms of pneumonia (a label of '0') or not (a label of '1'). If succesful, the resulting model would be used to assist medical professionals by expediting the diagnostic process. This project aims to develop an AI that would power a software tool that could be added to the medical proffesionals tool bag and help make diagnosing Pneumonia easier and faster. This would benefit both the medical providers and patients by reducing man-hours per diagnosis thereby reducing overall costs. This AI model is being developed independently by myself with the goal of being absorbed by a larger relevent medical service busines. Once the model is ready for beta testing, the next step will be to pitch the product to stake holders of interested businesses who would have the resources develop an industry grade UI for the product and robust UX research.  

# ![Business Use Case](images/presentation_assets/Business%20Use%20Case.png)  

Pneumonia is an infection of the lung that is quite prominent in the US today despite having some of the best medical care available. Pneumonia is the leading cause of death worldwide for children under 5 and is the most common cause for hospital admissions in the US, save for pregnancy<sup>[1](https://www.thoracic.org/patients/patient-resources/resources/top-pneumonia-facts.pdf)</sup>.  

It is clear that there is much room for innovation in the way we handle the scourge Pneumonia, however the medical care *system* is notoriously complex and slow to adapt to the rapidly evolving technological landscape. Instead of reinventing the wheel, it is best to leverage extant industry technology and processees that capture Pneumonia among other diseases. Due to the ubiquitious use of x-ray technology around pneumonia and other diseases of the chest and torso, a wealth of image data is available that demonstrates quality examples of both healthy (or at least without pneumonia) respiratory tracts and those, in fact, with pneumonia.  

A computer vision based machine learning model (ML), once properly trained and optimized, could easily be folded into the afformentioned existing technology. By partnering with the software providers behind the existing technology used to capture, view and store x-ray data. Said ML could easily be incorporated into the software and its *prediction* be provided to the medical profesional at their discretion. So long as clear and concise documentation is provided to the medical profesionals regarding how best to utilize and interpret this new AI feature of their pre-existing software this could potentially expedite the diagnostic timeline and provide the care provider with greater confidence in their own diagnoses.

In other words, the business goal of this project is to be absorbed by an existing software provider that can integrate this AI model as the underlying engine for a new tool and/or feature in their existing, deployed software. The benefit to said software provider is to circumvent the AI development process and focus on areas they are already expert at such as UI/UX, backend, database, etc. This allows the software provider to further improve their product, without pioneering into totally new territory; likewise allows myself as a datascientist to make a meaningful and profitable impact without the cost of learning or hiring for a litany of adjacent skill sets.  

# ![Data Source](images/presentation_assets/Data%20Overview.png)
This project utilizes a dataset of 5,856 jpeg images of chest x-ray scans from a women's and children's hospital in Guangzhou, China. As shown in the [EDA notebook](eda.ipynb), class balance for each sample split is as follows: Training set is 74% true positives, test set is 63% true positives, and validation set is 50% true positives. The training sample contains 5216 images, the test contains 624 images, and the validation set contains 16. 

| Normalized Class Balance | Relative Sample Sizes |
| ------------- | ------------- |
|![class balance plot](images/normalized_sample_class_balance.png)|![Class balance pie](images/pie_sample_class_balance.png)|


This dataset originally comes from [Mendeley Data](https://data.mendeley.com/datasets/rscbjbr9sj/2) and it was directly sourced for this project from [Kaggle.com](https://www.kaggle.com/datasets/paultimothymooney/chest-xray-pneumonia) using the kaggle API on the command line. The original kaggle post describes the dataset as follows:

> The dataset is organized into 3 folders (train, test, val) and contains subfolders for each image category (Pneumonia/Normal). There are 5,856 X-Ray images (JPEG) and 2 categories (Pneumonia/Normal). Chest X-ray images (anterior-posterior) were selected from retrospective cohorts of pediatric patients of one to five years old from Guangzhou Women and Children’s Medical Center, Guangzhou. All chest X-ray imaging was performed as part of patients’ routine clinical care. For the analysis of chest x-ray images, all chest radiographs were initially screened for quality control by removing all low quality or unreadable scans. The diagnoses for the images were then graded by two expert physicians before being cleared for training the AI system. In order to account for any grading errors, the evaluation set was also checked by a third expert. 

# ![Data Preparation](images/presentation_assets/Data%20Prep.png)
![img](images/presentation_assets/xray%20examples.png)

As mentioned above the data comes already split into three distinct samples (Train, test, and validate). Additionaly the originators of the data have already performed quality control and binary class labeling by qualified professionals. There was no use of resampling methods throughout this project due to the sufficient volume of data (i.e. even the minority class is composed of several thousand data points). Therefore the only data preparations steps left were to prepare the image files to be "readable" to a Keras CNN, this was achieved using the Keras [ImageDataGenerator](https://www.tensorflow.org/api_docs/python/tf/keras/preprocessing/image/ImageDataGenerator) class along with its related flow_from_directory() method to read the jpg files into the generator. Upon instantiation of the generator objects the data is normalized so that all first order tensors are scaled 0-1 instead of 0-255 (standard pixel values), and is set to grayscale so that each image is represented by a two dimensional tensor with a single color channel instead of three.

One key advantage to utilizing the ImageDataGenerator class is that it allows the X and y data to be readable by other Keras function and methods with a single variable. The generator objects all share the same parameters; the color mode is set to grayscale, reducing unnecessary volume and complexity of the input data, the class mode is set to binary, reflecting the binary classification requirements of this project, images are resized to 150 by 150 pixels, and the batch size was originally 16 but then reduced to 8 as it appeared to improve the models ability to learn. 
 
For more details on the data and the preprocessing steps take please review the [EDA notebook](eda.ipynb) 

# ![Modeling Methods](images/presentation_assets/Modeling%20Method.png)
The modeling process, demonstrated in detail in the [Modeling notebook](modeling.ipynb), consisted of first building a baseline model with a lean and arbitrary architecture and training protocol, and then measuring more complex and sophisticated iterations against the initial basline model. The measures used primarily were recall and F1-score.

## Model Architecture
The first major iteration was experimenting with the architecture of the model itself. I experimented with different amounts and patterns of convultional, max pooling, and dense layers. Finally arriving at a fairly simple architecture but still more complex than the base model.

| Baseline Model Summary       | Final Model Summary      |
| ------------- | ------------- |
|<img src="images\base_model_summary.png">|<img src="images\final_model_summary.png">|

## Model Training Protocol
The next major iteration was experimenting with more robust training parameters (specifically within .fit() function). Various arrangements of steps per epochs, total number of epochs and validation steps per epoch were all experimented with. Considering the model's strong tendency to overfit it was ensured that the training protocol remained such that every training data point was seen by the model only once. At this stage epoch counts were tested in multiples of five from 5 to 25. Eventually 15 epochs was selected as the ideal training protocol. 

#### 10 Epoch Training History
![base history](images/ten_epoch_history.png)
#### Final Model Training Histroy
![final history](images/final_model_history.png)



## Network Regularization
The third major phase of model development was experimenting with various regularization techniques. The least effective of which was dropout regularization (dropping out random nodes from certain layers), this technique made the key performance indicators (KPIs) extremely eradic and not obviously converging on any values or approaching any limits (i.e. lots of zig zigs, not a lot of curves). Hoever L2 regularization showed clear benefits and after some experimentation a weight of 0.005 showed to be the most optimal.  

| L2 Regularization       | Dropout Regularization      |
| ------------- | ------------- |
|<img src="images\L2_matrix.png">|<img src="images\dropout_matrix.png">|



## Optimization Algorithm
The fourth major iteration was experimenting with various optimization algorithms. Up until this point in the project there had been none specified, and without sepcifying an algorithm the learning rate can not be adjust (that I can find). After testing three different algorithms, [stochastic gradient descent](https://keras.io/api/optimizers/sgd/), [adaptive momentum estimation](https://keras.io/api/optimizers/adam/), and [adaptive delta](https://keras.io/api/optimizers/adadelta/), it was discoverd that Adam (adaptive moment estimation) was the best algorithm for this model and this project.  Adadelta was so terrible that it predicted everything as pneumonia so its visualization is not show below, only that for SGD and Adam.

| SGD       | Adam      |
| ------------- | ------------- |
|![SGD](images/SGD_class_balance_prediction.png)|![Adam](images/Adam_class_balance_prediction.png)|  

## Learning Rate
The fifth and final iteratation of model development was experimenting with various learning rates of the Adam optimization algorithm. After some experimentation it was determined that the optimal learning rate using Adam is .001. Examples of each learning rate tested are not available in the modeling notebook, because I re-ran the exact same at various learning rates and took note of the KPIs, this was in the interest of the readibility of the notebook and timeliness of the overall process. The learning rate parameter can be easiliy located in the "Learning Rate" section of the modeling notebook and can be easily changed and re-ran if you are so-inclined. 

|     Results at 1e-3 Learning Rate   |       |
| ------------- | ------------- | 
|![class balance prediction](images/learning_rate_class_balance_prediction.png)|![confusion matrix](images/learning_rate_matrix.png)| 

# ![Model Evaluation](images/presentation_assets/Model%20Evaluation.png) 
The key performance indicators utilized for this project are primarily recall, and loss. F1-score is consdered to provide context to the dynamic between recall and precision while accuracy is considered to provide context to the dynamic between recall and loss. In the medical industry it is prefered to have higher rate of false positives and minimal risk of false negatives, rather than a high accuracy with a relatively higher false negative rate. Basically its assumed that false positives will be cross examined by other diangostic tools and eventually caught where as in the case of a false negative a patient may be sent out the door who is in fact ill and this diagnoses never gets cross examend. The performance of the final model on both the test and validation data I find very satisfactory in regard to these unique needs.  

For the sake of readibility and diagnostic efficiency I wrote a small module made custom for this project that provides several functions useful for generating performance statistics such as recall and loss, and rendering visuals of those performance statistics like confusion matrices and visualizing training history. The source code for this module can be found in the [project toolkit](project_toolkit.py) py file found in the root diretory of this repository.

The model had a tendency to overfit throughout the project, made evident by bizarre results such as 1.0 recall and 7.84 loss, and an accuracy of  0.6. A result like this I interpreted as an indication that the model is correctly classifying all the cases of pneumonia but also miss classifying *a lot* of truly negative cases as positive. 

In essence a false positive rate greater than zero is okay for the end user so long as there are not so many false positives that it becomes untenable for the end user to cross-examine and verify with other diagnostic tools. This could potentially lead to the tool being more of a burden than an aid.

So my efforts throughout the iterative process were focused on analyzing these KPIs and taking further steps to reduce the loss function as much as possible while mainting the recall as close to 1.0 as possible. In the end, the final model captures essentially all true cases of pneumonia and predicts false positives on about 20% of the test data and  true negatives on about 20%; on the validation data it correctly predicts all 16 cases. This still has a lot of room for improvement, but compared to the protypical variations of the model it certainly moves in the direction of this projects goals and, more or less, achieves what it is meant to.

## Final Model Diagnostic Visuals  
#### **Final Model Training History:**  
![final model history](images/final_model_history.png)
#### **Final Model Classification Performance:**  
|| Predicted vs True Class Balance  | Confusion Matrix  |
|----|-----|-----|
|Test Data|![final prediction balance](images/final_class_balance_prediction.png)|![final matrix](images/final_confusion_matrix.png)|
|Validation Data|![final prediction balance](images/val_class_balance_prediction.png)|![final matrix](images/val_confusion_matrix.png)|

# ![Conclusion](images/presentation_assets/Conclusion.png)


## Final Model Description

The final version of the predictive model developed for this project is a convolutional neural network consisting ten total layers (eight hidden layers) using the Adam algorithm to optimize the gradient descent, a regularized network using L2 penalization and random partial dropout in the hidden dense layers. The key points of the model's architecture is as follows: 

	input layer
	1: Conv2D; 32 outputs, 3x3 window , relu activation, 150x150 input shape
	
	five hidden convolutional layers
	2: MaxPooling2D, 2x2 pool size
	3: Conv2D, 64 outputs, 3x3 window,  relu activation
	4: MaxPooling2D, 2x2 pool size
	5: Conv2D, 128 outputs, 3x3 window, relu activation
	6: MaxPooling2d, 2x2 pool size

	7: A flattening layer
	
	two hidden dense layers 
	8: Dense, 512 outputs, relu activation, Learning rate 0.0005, dropout 0.05
	9: Dense, 256 outputs, relu activation, Learning rate 0.005, dropout .01  
   
    output layer  
    10: Dense, 1 output, sigmoid activation

The input is essentially a 1D array where each element is a 2D array representing a single image, meaning all input images are grayscale. All data points are normalized from a scale of 0 - 255 to a scale of 0 - 1. The model is trained on the data in batches of 16 over 15 epochs, in such a way that the model sees every training image only once. 

## Reliability and Limitations

The model definitely predicts in a satisfactory mannor with the data currently available to this project, even data yet unseen. However because all the data originates from the same hospital, all that really tells us is that the model can generalize well to other data  *of the same kind and origin*. The origin is infact an individual women's and children's hospital in China. So we have to assume that the typical individual overwhelmingly represented by this dataset is east asian, young and female, and we have to assume that would incaulcate the model with some bias. Additionally there is a relatively significant class imbalance in both the training and test data; it probably has high fidelity with the population it actually represents, but perhaps not representative of the true class balance of observations it will be expected to analyze in a production setting. In other words the homogenous nature of the data origin, and the class imbalance of the source data both certainly invoke some bias or even naivete in the model. This would need to be addressed before releasing the tool to a production setting. All of these issues could be feasibly recrtified by a) soliciting data from similar hospitals on other continents and b) collecting data throughout beta testing.     

Furthermore there are hardware limitations. This projects has been conducted in its entirety on my personal machine, more sophisticated model's could be more quickly tested and iterated over with a more specialized computing solution. This could be resolved with a dedicated physical machine, or a cloud computing solution. If nothing else, as the dataset grows this would quickly become a necessity.   

## Recomendations moving forward
So, the very next steps steps I recommend are . . .  

    A: negotiate a partnership or buy out with an existing medical software provider  
    B: beta test with select medical providers  
    C: gather data even outside of beta-testing network   
    D: expand classification set to include bacterial vs varial pneumonia  
    E: expand model's ability to generalize to unseen data 


### Contact Me:
I deeply appreciate you taking the time to review this notebook and hope you have gained some value from this data science project.  
If you have any questions, suggestions or otherwise in need of contacting me you can find me through:
- [LinkedIn](https://www.linkedin.com/in/zeth-abney/)
- [Instagram](https://www.instagram.com/texan_space_cowboy/)
- zethusabney@gmail.com

If you are curious about Flatiron School or data science bootcamps in general you can see my entire bootcamp journey on my youtube channel [Data Cowboy](https://www.youtube.com/channel/UCkYdKUId0iITN_czJ0X-sbA)

# Repository Stucture
├── [data](data) (x-ray images organized by class into subdirectories, gitignored)  
├── [images](iamges) (jpg and png files used for presentation and aesthetics)  
├── [presentation](presentation) (contains pptx files used for non-technical presentation, and pdf files for project submission, gitignored)     
├── [eda.ipynb](eda.ipynb) (exploratory analysis, previews data and tests modeling conventions)    
├── [modeling.ipynb](modeling.ipynb) (contains all interations of the model and the respective analysese)   
├── [project_toolkit.py](project_toolkit.py)     
└── README.md  