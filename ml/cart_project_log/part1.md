## Problem formulation:

Classify photos at a gate to a facility. Identify cars/trucks and front/back of cars.

### Weekend one: Manual classification

 - Created a quick tool for manual data classification. (React+Konva and Sinatra+SQLite).
 - Manually classified 1100 images.
 - Gave up on http://mlcourse.ai. Not enought time to do the homeworks. Will concentrate on finishing the specialization on Coursera first https://www.coursera.org/specializations/deep-learning.
 
 
### Weekend two: Initial learning

 - Created Jupyter notebook where data from manual classification (csv) are loaded,
 - Loaded image data to the Jupyter notebook, display data, etc.
 - Copied several CNN models in Keras, tried one by one.
 - Trained a model with ~15 Keras elements, convolutional layers, dropout, cathegorical cross-entropy cost function, used Adam optimizer with default values for hyperparameters.
 
 ![enter image description here](https://lh3.googleusercontent.com/tDu_pz8ZSVa2tGT74Ch-qenu-_Id-ogR14E41z90DryS7_HUZO3_ERFUJ0rh7SE0ONSm-UoS2bGr9A "Initial mode accuracy")
 
- Accuracy train/test >90/>60, large variance. Seems the capacity of the model is more that sufficient. Probably need more training data.

- Inspiration: take a look at keras image processing and feeding into model for training. Carry on with Coursera Sequential models course.

### Week three: Improvements

 - Prepared (a bit) more of training data.
 - Looking at how to deploying models as Python service: Flask
 -  Experimantaion with GPU learning nvidia 1050, 2GB RAM, out of memory even with bach_size = 1 on my home desktop.

Should I consider some (systematic) hyperparameter search? Or just get more data?

#### Experiment with varying "decay" for Adam

To address oscilation in test set accuracy I tried to limit the learning rate by the learning decay parameter passed to the Adam optimizer.

##### Default setting: decay=0
Significant oscilation of the test set accuracy.

![](https://lh3.googleusercontent.com/t-wdzVXcntBQLInHL-kQuUflpCtbHvipdNNr6W_t_BMg4MpYG0hnYRt_zj2jAzRQ1AUmrrHOGFUlVg "decay=0")

##### decay=0.01
The oscilation is limited, but the train accuracy dropped significantly. Test accuracy did not improve compared to the best values from the previous experiment.

![](https://lh3.googleusercontent.com/CQzuVf_TqrepfVnw01Z_FP96tEZaAd80EcI0WOi0yRqKKftgCCJtsKW9wHU7A4Im9C14lMlWdjroKQ "decay=0.01")


##### decay=0.005

With this middle setting the oscilation was minimized and at the same time both train and test accuracy are slightly better.

![](https://lh3.googleusercontent.com/GEy78V5rFWpKcxu6VM2stUt9PKNw_zHuJuOfWMGrCBD4Y3OnE7g_TT816xq1j7JYcCUhiaJbgvQRCA "decay=0.005")
