# Grayscale image colorization

Course project of [Machine Learning for Finance](https://github.com/PHBS/2018.M1.MLF) at [PHBS](http://english.phbs.pku.edu.cn/). This project uses convolutional neural network (CNN) to colorize grayscale images. The
repository is where we develop model and present briefly the result. We welcome contributions if you are interested in our
project. For example, you can:

* [Submit bugs or issues](https://github.com/devon-ge/PHBS_MLF_2018/issues) to improve the performance of our model.
* [File pull requests](https://github.com/devon-ge/PHBS_MLF_2018/pulls) if you have better ideas.

## Team Members

* [Ge Desheng](https://github.com/devon-ge), student ID: 1701213756
* [Wang Yumeng](https://github.com/yumengwang123), student ID: 1701213112
* [Wu Gan](https://github.com/SuperWGAaron), student ID: 1701212974
* [Zhang Mingyu](https://github.com/myzhangcn), student ID: 1701213151

## Motivation

With the popularity of machine learning, a variety of applications are hoping to simplify
both our lives and jobs. The state-of-the-art machine learning methods in pattern recognition
enable humans to find the intrinsic relationship of things. For example, image recognition often
compares the gray scale of scanned picture with dataset for identificaiton. How to transform a
colorful picture to a grayscale one attracts attentions in algorithm research. This project, however,
intends to colorize a grayscale picture, i.e., regain the original image. We try to map grayscale to RGB
colors acorrding to gray scale distribution. Colorful pictures contains more
information (such as RGB pixels), thus contributing to better recognition. Also, we can apply this
colorization algorithm to repair old pictures.

![flowchart](test/flowchart.png)

## Data and preprocessing

We obtian images of natural landscape by a spider. The pictures are classified by principal components analysis (PCA). convert original images to standardized 256 by 256 Lab images. The model receives colorful images and load them in grayscale. The neuron network then trains the samples and outputs colorized images.

### Images spider

Spider for this project are implemented under the [`Scrapy`](https://scrapy.org/) framework. Roughly speaking, a `Scrapy` project is nothing but a folder that contains auto-generated files (incluing `items.py`, `middlewares.py`, `pipelines.py` and `settings.py` under `PROJECT_NAME` folder, and spider(s) under `spiders` folder).

Among these, we mainly config in `items.py` and `settings.py`. [`items.py`](tooopen_img/tooopen_img/items.py) confines the crawling field (the spider only extract fields defined in this file). [`settings.py`](tooopen_img/tooopen_img/settings.py) contains project-specific configurations. More info on [Scrapy Tutorial](https://docs.scrapy.org/en/latest/intro/tutorial.html).

Pictures are crawled by the spider [`tooopen`](tooopen_img/tooopen_img/spiders/tooopen.py
) from the [nature category of **Toopen**](http://www.tooopen.com/img/87.aspx) (A website that posts categorized pictures). All pictures are name by the SHA1 hash of its url and stored in [`images/full`](images/full) directory (default of `Scrapy`).

To reproduce the crawling, just clone this repository and run spider `tooopen` via `Scrapy` command in terminal.

1. Clone this repository to your local disk

    `$ git clone https://github.com/devon-ge/PHBS_MLF_2018.git`

2. Change to `tooopen_img` directory (root directory of this scrapy project, should contain `scrapy.cfg` and this `README.md`)

    `$ cd PHBS_MLF_2018/tooopen_img`

3. Run `$ scrapy crawl tooopen[ -s CLOSESPIDER_ITEMCOUNT=60]`. The terminal should show the logs of crawling requests/response.

Now, you can view the pictures in [`images/full`](images/full) (a folder automatically generated by
scrapy under a project's root directory).

**Notes**: In step 3, `60` is the maximum number of item requests (can be set per the demand).
The spider will keep running until all pictures are crawled if we
omit the `CLOSESPIDER_ITEMCOUNT` option. That's time consuming!

Below are some examples:

![example1](images/full/3d653a61eb0a12562e17636b17144418097b8af1.jpg) ![example2](images/full/fb5f301c86b8e948cdb68a2e273fea24cdb8cdb1.jpg)

![example3](images/full/6cc751a4d4cde64e164df5a93b3df2ccceaef38c.jpg) ![example4](images/full/2d773493dc2415b631a66adc135e86c88e88fc03.jpg)

### A simple classifier

Once we get the data (images), we can start training the neuron network. To improve the accuray, we first build a classifier to classify pictures. For example, mountains and rivers pictures should be divided into separated groups. In this way, the model captures category-dependent features and thus should theoretically perform better than mixed training.

We use PCA score of RGB to represent each picture and then use k-means to cluster those images. Each time we will pick one group to create the trainning and test set. Thus make sure trainning and test images are similar to a certain degree without the extra effort of classification by human. In the end we train our model with 110 samples in [`images/full`](images/full).

### Color channels conversion

First, to reduce computational intensity and improve the efficiency of the model, RGB images are resized to 256 by 256. Then, we convert the image format from `RGB` to `Lab`. Below are details of the conversion process.

* Convert color channels of images from `RGB` to `Lab`. In `Lab` format, `L` is lightness, ranging from 0(black) to 100(white), `a` is the green/red channel, ranging from -128 to 127, and `b` is the blue/yellow channel, ranging from -128 to 127. The conversion from `RGB` to `Lab` is invertible. It is easier to train two color channels in Lab than three channels in RGB, and `a` and `b` are uncorrelated.

* The convolution neural network inputs grayscale images (images with the L channel), and outputs the other two color channels, i.e., a and b. Then, L, a, and b are concatenate together, and full color images are retrieved. Last, images are converted from `Lab` back to `RGB`.

:octocat:|Raw|Compressed (width=256 px)
---|---|---
Gray|![Raw picture](./test/example_Gray.jpg) | ![Compressed picture](./test/com_example_Gray.jpg)
RGB|![Raw picture](./test/example_RGB.jpg) | ![Compressed picture](./test/com_example_RGB.jpg)

* Images are divided into test sets and training sets by a certain proportion. In this project, both training set and test set are standardized pictures. [`images/full1`](images/full1) contains 110 training samples (standardized from [`images/full`](images/full) ). [`images/test1`](images/test1) contains 10 test samples (standardized from [`images/test`](images/test))

## Neural network training

We use a complicated neural network that contains more than 10 layers. In the first half layers, we decompose a standardized picture into small ones (pixels decomposition). The minimal unit is 32 by 32 object. The other half layers return the original size by upsampling. The activation function in the hidden layers is `ReLU`. Below is the flowchart of how this neural network works.

### CNN structure

![CNN structure](test/CNN_Structure.png)

For a trial version of incorporating part of ResNet18 into the CNN for automatic colorization, please visit (https://github.com/SuperWGAaron/MLF_Grayscale_Image_Colorization).

### Build CNN model based on Keras

* The picture below shows the process of some popular optimizers intuitively. (eg. SGD, Momentum, NAG, Adagrad, Adadelta, Rmsprop)
* In our model, we use Rmsprop method, which is an adaptive learning rate method and proposed by Geoff Hinton.

![example1](./test/optimizer.gif)

*Source*: [*CS231n*](http://cs231n.github.io/neural-networks-3/)

### Result

The following table shows the colorizing result with different training epochs. The coloraziton is evidently better when we increase the trainning times. The difference between 1000-epoch model and 2000-epoch model is subtle. When increasing the epoch to 5000, the model yields less error compared to the 2000-epoch one. The colorization difference between 2000-epoch and 5000-epoch, however, is negligible. The model in run on Windows 7 ultimate with GTX 1080 Ti.

Epoch|1|2|3|4|5|6|7|8|9|10
---|---|---|---|---|---|---|---|---|---|---
Gray|![1](./Gray2Lab/result_gray/img_0.png) |![2](./Gray2Lab/result_gray/img_1.png)|![3](./Gray2Lab/result_gray/img_2.png)|![4](./Gray2Lab/result_gray/img_3.png)|![1](./Gray2Lab/result_gray/img_4.png)|![1](./Gray2Lab/result_gray/img_5.png)|![1](./Gray2Lab/result_gray/img_6.png)|![1](./Gray2Lab/result_gray/img_7.png)|![1](./Gray2Lab/result_gray/img_8.png)|![1](./Gray2Lab/result_gray/img_9.png)
50|![1](./Gray2Lab/result_50/img_0.png) |![2](./Gray2Lab/result_50/img_1.png)|![3](./Gray2Lab/result_50/img_2.png)|![4](./Gray2Lab/result_50/img_3.png)|![1](./Gray2Lab/result_50/img_4.png)|![1](./Gray2Lab/result_50/img_5.png)|![1](./Gray2Lab/result_50/img_6.png)|![1](./Gray2Lab/result_50/img_7.png)|![1](./Gray2Lab/result_50/img_8.png)|![1](./Gray2Lab/result_50/img_9.png)
1000|![1](./Gray2Lab/result_1000/img_0.png) |![2](./Gray2Lab/result_1000/img_1.png)|![3](./Gray2Lab/result_1000/img_2.png)|![4](./Gray2Lab/result_1000/img_3.png)|![1](./Gray2Lab/result_1000/img_4.png)|![1](./Gray2Lab/result_1000/img_5.png)|![1](./Gray2Lab/result_1000/img_6.png)|![1](./Gray2Lab/result_1000/img_7.png)|![1](./Gray2Lab/result_1000/img_8.png)|![1](./Gray2Lab/result_1000/img_9.png)
2000|![1](./Gray2Lab/result_2000/img_0.png) |![2](./Gray2Lab/result_2000/img_1.png)|![3](./Gray2Lab/result_2000/img_2.png)|![4](./Gray2Lab/result_2000/img_3.png)|![1](./Gray2Lab/result_2000/img_4.png)|![1](./Gray2Lab/result_2000/img_5.png)|![1](./Gray2Lab/result_2000/img_6.png)|![1](./Gray2Lab/result_2000/img_7.png)|![1](./Gray2Lab/result_2000/img_8.png)|![1](./Gray2Lab/result_2000/img_9.png)
5000|![1](./Gray2Lab/result_5000/img_0.png) |![2](./Gray2Lab/result_5000/img_1.png)|![3](./Gray2Lab/result_5000/img_2.png)|![4](./Gray2Lab/result_5000/img_3.png)|![1](./Gray2Lab/result_5000/img_4.png)|![1](./Gray2Lab/result_5000/img_5.png)|![1](./Gray2Lab/result_5000/img_6.png)|![1](./Gray2Lab/result_5000/img_7.png)|![1](./Gray2Lab/result_5000/img_8.png)|![1](./Gray2Lab/result_5000/img_9.png)
Raw|![1](./images/test1/e4ddee715ad3e89e02ce705bd79eb75515115031.jpg) |![2](./images/test1/e55e357725603349c61d2760738fde41e9dce196.jpg)|![3](./images/test1/e8e9bbcd94ed1b011456d4560e24bdd9fef995a6.jpg)|![4](./images/test1/ee693c631afc69f3314a67f1c38573e290d20d12.jpg)|![1](./images/test1/efde62f1219af53549c32f4d303a93afba684020.jpg)|![1](./images/test1/f01d30750b0b5ef78bd0e835e8713ff461f42f70.jpg)|![1](./images/test1/f0748fe286f542b976c03f5aa81ef1f8d2c9a0ff.jpg)|![1](./images/test1/f30e5fda9409e6fd4f263f6a77f6878f09504104.jpg)|![1](./images/test1/f76a42e869ef9f4ee547f45e78f52e20b102b9b2.jpg)|![1](./images/test1/ff892b1cf137703a71201a2444645e074c826b5e.jpg)

The subtle improvement of this CNN model is highly relied on the convergece speed. Below shows the decreasing of loss when the epoch of training increase. After about 1500 epochs of training, the loss implied by this model decreases increasingly slow.

![loss](/Gray2Lab/loss_2000.png)

## Conclusion

In this project, we develop a convolutional neural network to colorize gray pictures. The CNN models shows efficacy from the perspective of research purpose. The computationally intensive model shows predictably better performance with the increase of training epochs. The improvement, however, diminishes as we increase the epoch. Such effect is evident after about 1500 epochs of training.

Naturally the limit of loss derives from the trainig set, and the model specificaiton. We argue that model specification accounts for the convergence of loss function with regard to epochs of training. Although large epoch (e.g., 5000) will absolutely yield less loss, it's computationally intensive and beyond ability of the human eye identification, especially in our project. 2000 performs good that balances model predictability and computational intensity.

## Improvement in the future

1. The performance of classfier can be improved in the future, more high quality samples and training times will improve the performance. Using the classify network trained by google is a potential way to improve.

2. Loss function is another thing that can be improved. Current loss function is "MSE", "MAE". Considering the unsupervised network loss and the classification loss.

3. CNN topological structures and inner parameters can be improved by using  optimization algorithm.

4. In order to get a better performance, we can add mannual operations (eg. mark some pixs with specific colour which is decided by human and so on)

## Reference

Zhang, R., Isola, P., & Efros, A. A. (2016, October). Colorful image colorization. In European Conference on Computer Vision (pp. 649-666). Springer, Cham.
