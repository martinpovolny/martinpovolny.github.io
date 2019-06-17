## June

A number of experiments from the hyperparameter search gave over 80% validation accuracy, however it did not get me over 90%.

### Data augmentation

So as a next step I implemented data augmentation. I extended the `ImageGenerator` with data augmentation and I also tried working with smaller images.

With these changes I got just a bit over 90% validation accuracy.

### Transfer learning

As a next experiment I followed an example from the Convolutional Neural Networks in TensorFlow course and implemented a transfer learning training example with pre-trained **Inception V3** model.
For that I created a training se with square data to match what the pre-treained model was used for.

Early experiments with this give me over 80% accuracy, but not much higher.
