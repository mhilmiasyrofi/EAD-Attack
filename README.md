# Reproducing EAD-Attack using Docker

**by Muhammad Hilmi Asyrofi**

pull required docker images to prepare Tensorflow 1.3.0 with Python3 on GPU
```
docker pull tensorflow/tensorflow:1.3.0-devel-gpu-py3
```

run docker container
```
docker run --name eadattack --rm --gpus '"device=1"' -it -v ~/Documents/EAD-Attack/:/root/EAD-Attack/ tensorflow/tensorflow:1.3.0-devel-gpu-py3
```
```
cd /root/EAD-Attack/
```

install Keras with specific version (because the Tensorflow 1.3.0 was last maintained 3 years ago, thus we need to install keras around 3 years ago version). I tried 2.1.2 and it works
```
pip3 install keras===2.1.2
pip3 install h5py
```

Read the paper description and run the instruction bellow

#### Due to library depreceation, we need to change some implementation

**1. `scipy.misc.imresize` depreciation**

we need Pillow library
```
pip install Pillow
```

change `setup_inception.py` line 259
```
dat = np.array(scipy.misc.imresize(scipy.misc.imread(image),(299,299)), dtype = np.float32)
```
into
```
dat = np.array(Image.fromarray(scipy.misc.imread(image)).resize((299,299)), dtype = np.float32)
```

## Train model

Train defensively distilled MNIST and CIFAR-10 models under specified temperatures:

```
python3 train_models.py -dd -t 1 10 100
```

## Run attack
Save original and adversarial images in the saves directory

```
python3 test_attack.py -sh
```

The adversarial image (for mnist) is save inside `saves/mnist/EN` folder



EAD: Elastic-Net Attacks to Deep Neural Networks 
=====================================

EAD is a **e**lastic-net **a**ttack to **d**eep neural networks (DNNs).  
We propose formulating the attack process as a elastic-net regularized optimization problem, featuring an attack which produces L1-oriented adversarial examples which includes the state-of-the-art L2 attack (C&W) as a special case. 

Experimental results on MNIST, CIFAR-10, and ImageNet show that EAD yields a distinct set of adversarial examples and attains similar attack performance to state-of-the-art methods in different attack scenarios. More importantly, EAD leads to improved attack transferability and complements adversarial training for DNNs, suggesting novel insights on leveraging L1 distortion in generating robust adversarial examples. 

For more details, please see our paper:

[EAD: Elastic-Net Attacks to Deep Neural Networks via Adversarial Examples](https://arxiv.org/abs/1709.04114)
by Yash Sharma\*, Pin-Yu Chen\*, Huan Zhang, Jinfeng Yi, Cho-Jui Hsieh (AAAI 2018)

\* Equal contribution

The attack has also been used in the following works (incomplete):

[Attacking the Madry Defense Model with L1-based Adversarial Examples](https://arxiv.org/abs/1710.10733)
by Yash Sharma, Pin-Yu Chen (ICLR 2018 Workshop)

[On the Limitation of Local Intrinsic Dimensionality for Characterizing the Subspaces of Adversarial Examples](https://arxiv.org/abs/1803.09638) by Pei-Hsuan Lu, Pin-Yu Chen, Chia-Mu Yu (ICLR 2018 Workshop)

[Bypassing Feature Squeezing by Increasing Adversary Strength](https://arxiv.org/abs/1803.09868)
by Yash Sharma, Pin-Yu Chen

[On the Limitation of MagNet Defense against L1-based Adversarial Examples](https://arxiv.org/abs/1805.00310)
by Pei-Hsuan Lu, Pin-Yu Chen, Kang-Cheng Chen, Chia-Mu Yu (IEEE/IFIP DSN 2018 Workshop)

The algorithm has also been repurposed for generating constrastive explanations in:

[Explanations based on the Missing: Towards Contrastive Explanations with Pertinent Negatives](https://arxiv.org/abs/1802.07623)
by Amit Dhurandhar, Pin-Yu Chen, Ronny Luss, Chun-Chen Tu, Paishun Ting, Karthikeyan Shanmugam and Payel Das (NIPS 2018)

The experiment code is based on Carlini and Wagner's L2 attack.  
The attack can also be found in the [Cleverhans Repository](http://cleverhans.readthedocs.io/en/latest/_modules/cleverhans/attacks.html#ElasticNetMethod).


Setup and train models
-------------------------------------

The code is tested with python3 and TensorFlow v1.2 and v1.3. The following
packages are required:

```
sudo apt-get install python3-pip
sudo pip3 install --upgrade pip
sudo pip3 install pillow scipy numpy tensorflow-gpu keras h5py
```

Prepare the MNIST and CIFAR-10 data and models for attack:

```
python3 train_models.py
```

To download the inception model (`inception_v3_2016_08_28.tar.gz`):

```
python3 setup_inception.py
```

To prepare the ImageNet dataset, download and unzip the following archive:

[ImageNet Test Set](http://jaina.cs.ucdavis.edu/datasets/adv/imagenet/img.tar.gz)


and put the `imgs` folder in `../imagesnetdata`. This path can be changed
in `setup_inception.py`.

Train defensively distilled models
-------------------------------------

Train defensively distilled MNIST and CIFAR-10 models with temperature varying from 1 to 100:

```
python3 train_models.py -dd
```

Train defensively distilled MNIST and CIFAR-10 models under specified temperatures:

```
python3 train_models.py -dd -t 1 10 100
```

Run attacks
--------------------------------------

A unified attack interface, `test_attack.py` is provided. Run `python3 test_attack.py -h`
to get a list of arguments and help. Note the default values provided as well. 

To generate best-case, average-case, and worst-case statistics, add "-tg 9" to command.

For computational efficiency, maximize the batch size and fix the 'initial_constant' to a large value, setting the number of binary search steps to 1.

The following are some examples of attacks:

Run the L1-oriented attack on the Inception model with 100 ImageNet images

```
python3 test_attack.py -a L1 -d imagenet -n 100
```

Run the EN-oriented attack on the defensively distilled (T=100) CIFAR model with 1000 images

```
python3 test_attack.py -d cifar -tp 100
```

Save original and adversarial images in the saves directory

```
python3 test_attack.py -sh
```

Generate adversarial images on undefended MNIST model with confidence (50), attack defensively distilled (T=100) MNIST model

```
python3 test_attack.py -cf 50 -tm dd_100
```

Adversarial Training
-------------------------------------

Adversarially train MNIST models by augmenting the training set with L2, EAD(L1), EAD(EN), L2+EAD(L1), and L2+EAD(EN)-based examples, respectively. This will use the provided numpy save files in the train directory.

```
python3 train_models.py -d mnist -a
```

Generate and save your own training set examples for use in adversarial training (ex - L1-oriented attack)

```
python3 test_attack.py -a L1 -sn -tr
```

Now, attack an adversarially trained model (ex - L1-trained network)

```
python3 test_attack.py -adv l1
```
