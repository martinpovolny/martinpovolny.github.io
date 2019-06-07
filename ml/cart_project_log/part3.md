### May


### Part 1
 Got tensorflow 2 running (Fedora 29). Tried to get tensorflow 2 running also on windows, but so far w/o success. 
 Notes: need 2 packages from NVidia, most likely Visual Studio run-time, python 3.6 (not 3.7 for now it seems). 
 
 Created a script for learning using [Keras ImageDataGenerator](https://keras.io/preprocessing/image/), splitted data into directories by class and **fixed a number of misclassification**.
 With these changes it seems that I am getting much better results. (surely over 80, maybe 90% validation accuracy, need to properly measure to make sure).
 
 I also created a script that utilizes [Talos](https://github.com/autonomio/talos) for hyper parameter seach.
 Need to run it with the best model and interpret the results.
 
 Looking at 2 Coursera courses on Tensorflow 2. The theory is known to me and so far pretty shalow. But the hands on
 with Ternsorflow 2 and various interesing datasets are being introduced, that is interesting.
 [Introduction Tensorflow](https://www.coursera.org/learn/introduction-tensorflow/home/welcome) 
 and [Conv nets with Tensorflow](https://www.coursera.org/learn/convolutional-neural-networks-tensorflow/home/welcome).

### Part 2
 Influences by the Tensorflow courses I migrated the models I have from Keras with Tensorflow 1.x backend to the new Tensorflow 2.0 Keras interface. Got learning running on Google Colab using ImageDataGenerator.
 
Following https://www.tensorflow.org/tensorboard/r2/hyperparameter_tuning_with_hparams I created a notebook that uses Grid Search (really a bunch of nested `for` loops) to run 432 experiments searching for the model hyperparameters.

The experiments showed that with the 3.500 training examples verious (parametrized) versions of the model can get above 80% test accuracy in under 20 epochs.


### Next steps

 - Interpret results from latest experiments,
 - analyze missclassified examples (so far I postponed this because I found that I have errors in the training data, so I first fixed that),
 - deploy as a service.
 - add Talog to the Colab notebooks to play with more interesting hyperparameter search techniques
