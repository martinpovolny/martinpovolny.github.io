## June

A number of experiments from the hyperparameter search gave over 80% validation accuracy, however it did not get me over 90%.

### Data augmentation

So as a next step I implemented data augmentation. I extended the `ImageGenerator` with data augmentation and I also tried working with smaller images.

With these changes I got just a bit over 90% validation accuracy.

### Transfer learning

As a next experiment I followed an example from the Convolutional Neural Networks in TensorFlow course and implemented a transfer learning training example with pre-trained **Inception V3** model.
For that I created a training se with square data to match what the pre-treained model was used for.

Early experiments with this give me over 80% accuracy, but not much higher.

![Inception V3, 50 epochs](
https://lh3.googleusercontent.com/Px68hdy_RIYmTKhghwtlHrFzDV2zP-QbLBd5tJOdj2N3I9q52mUmC_mDC3d8vXrAzt1hStB8FiD4sLXINES9gTq1qdFsBqfZ2I3-PMJUzTsOYnJyKQNKpR8YhZ4uZ69aBXg_XaHyl-QINoN2_Npiue29cFn5M4pTQ07wSPQ7IDehob5Ox03n4S50shAe8xnXazZE2QEWjGGiuj7duxGsW9Vv8lX7DaKGfv0KMea6d9HJkbFq_BRjuF287b6ML25htsDK_C5tn_JHuWsA0iuHhcBzunlOOhgrKVdMyOm4G5oCkTrHiyxGiZjkWAwuU4iXxtEcVmeL39q9Nt_oYlxtBV1e1CnlC30MEbe_T_6TRENqPBLudYYoGI7qvVg3Kvle2iaMbddYWwkkOuLMNc7BKSnVFm8aAu5FOqpZjJUQH3UU5FeK2kROCMQ03fWUUtdXIObH0CU6syVvlz-TlBgtonYw1Yj0WbLzlJaz4QU3ShrbAQioKmbg7w19v9Xmu7yw3sRGLcJMwBjrpoZatemzVUlRtzN5PJJ04mAOSVPTABkD1kAtSB-ltnxstvsOtFF3EQp1zlMvIj9UB6mHOjpqN9PfVzNiw6Z0tHdzSSqXcoK7ghZM-RGq3p_MlxEvmF4trOBSTyAteHVqLnFzLUX6iECijyv8YzP-=w375-h264-no)
