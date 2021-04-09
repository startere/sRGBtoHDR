Based on the Siggraph Asia CNN paper + selectable bit depth.

Training of new weigths information and code is provided in the [training_code](training_code/) folder.

https://github.com/scikit-learn/scikit-learn/blob/main/doc/modules/outlier_detection.rst to be used for dataset cleaning after randomization

Dependencies:

* [TensorFlow](https://www.tensorflow.org/) for model specification and prediction with modified Softmax and biases (with my team basically discovered a way to pool the classification with controlled randomization, to be published when NDA terms are clarified).
* [TensorLayer](https://tensorlayer.readthedocs.io/en/latest/) for simplified TensorFlow layer construction.
* [OpenEXR](http://www.openexr.com/) in order to write reconstructed HDR images to disc.
* [NumPy](http://www.numpy.org/) and [SciPy](https://www.scipy.org/) for image handling etc.

```
$ pip install numpy scipy tensorflow tensorlayer OpenEXR
```

## Usage
1. Trained CNN weights for inference
2. Run `python hdrcnn_predict.py -h` to input options.

#### Example
Test images provided:

```
$ python hdrcnn_predict.py --params hdrcnn_params.npz --im_dir data --width 1024 --height 768
```

Prediction on individual frames:

```
$ python hdrcnn_predict.py --params hdrcnn_params.npz --im_dir data/img_001.png --width 1024 --height 768
``` 

The parameters trained for increased temporal coherence also use JPEG compressed images, so these are possible to use also for video with compression applied. There is some trade-of between reconstruction quality and temporal coherence. If you do not need to reconstruct video material, the [original parameters](http://webstaff.itn.liu.se/~gabei62/hdrcnn/material/hdrcnn_params_compr_regularized.npz) should be prefered.

#### Controlling the reconstruction
The HDR reconstruction with the CNN is completely automatic, with no parameter calibration needed. However, in some situations it may be beneficial to be able to control how bright the reconstructed pixels will be. To this end, there is a simple trick that can be used to allow for such control.

Given the input image **x**, the CNN prediction **y = f(x)** can be controlled somewhat by altering the input image with an exponential/gamma function, and inverting this after the reconstruction,

&nbsp;&nbsp;&nbsp;&nbsp;**y = f(x<sup>1/g</sup>)<sup>g</sup>**.

Essentially, this modifies the camera curve of the image, so that reconstruction is performed given other camera characteristics. For a value **g > 1**, the intensities of reconstructed bright regions will be boosted, and vice versa for **g < 1**. There is an input option `--gamma` that allows to perform this modification:

```
$ python hdrcnn_predict.py [...] --gamma 1.2
```
In general, a value around *1.1-1.3* may be a good idea, since this prevents underestimation of the brightest pixels, which otherwise is common (e.g. due to limitations in the training data).
