# Figure 3 Reproduction and Dataset Extension

## 1. Overview

This class project reproduces Figure 3 of [“Deep Neural Networks as Gaussian Processes”](https://arxiv.org/abs/1711.00165) by Lee et al. (ICLR 2018). The experiment uses a neural network Gaussian process (NNGP) kernel to compare predictive variance with actual squared prediction error. Test examples are sorted by predicted variance and averaged in groups of 100; each plotted point represents one group. Pearson correlation between the binned variance and error is reported separately for Tanh and ReLU.

## 2. Reproduction: Original vs. Ours

The comparison below uses our MNIST run with `num_train=1000`. Its correlations are 0.9890 for Tanh and 0.9870 for ReLU. The paper’s original Figure 3 is available in the [paper](https://arxiv.org/abs/1711.00165). Both show a strong positive relationship between predicted variance and error, matching the paper’s qualitative claim that NNGP predictive uncertainty tracks prediction error.

| Paper: original Figure 3 | Our MNIST reproduction, 1,000 training examples |
|---|---|
| [View the paper and Figure 3](https://arxiv.org/abs/1711.00165) | ![Our MNIST Figure 3 reproduction](uncertainty_fig3_mnist.png) |

## 3. Reproduction Instructions

Clone the project, build its TensorFlow 1.15 Docker image, and start the interactive container from the project directory:

```bash
git clone https://github.com/SharonBayela/DeepNeuralNetworkGP.git
cd DeepNeuralNetworkGP
docker build --platform linux/amd64 -t DeepNeuralNetworkGP-project .
docker run --platform linux/amd64 -it -v "$(pwd)/output":/DeepNeuralNetworkGP/output DeepNeuralNetworkGP-project
```

## 4. Extension: Written Description

This project adds Fashion-MNIST and CIFAR-100 loaders alongside the original MNIST and CIFAR-10 datasets. The same NNGP hyperparameters (`depth=3`, `weight_var=2.0`, and `bias_var=0.2`) and both nonlinearities (Tanh and ReLU) were used for every dataset. Each dataset was run with 1,000 and 500 training examples, producing eight plots. The extension tests whether Figure 3’s uncertainty–error relationship persists across image datasets and training set sizes.

Fashion-MNIST is a same-format substitute for MNIST: both contain 28×28 grayscale images in 10 classes. This holds image dimensions and class count fixed while changing image content. CIFAR-100 is a same-total-size substitute for CIFAR-10: both contain 60,000 32×32 RGB images. CIFAR-100 spreads that pool across 100 classes rather than 10, giving approximately 600 images per class compared with CIFAR-10’s approximately 6,000, a tenfold reduction in per-class data. It therefore tests the relationship under a much sparser per-class signal while keeping overall dataset scale and image format fixed.

The correlations between binned predictive variance and binned squared error are:

| Dataset | Train examples | Tanh correlation | ReLU correlation |
|---|---:|---:|---:|
| MNIST | 1,000 | 0.9890 | 0.9870 |
| MNIST | 500 | 0.9772 | 0.9733 |
| Fashion-MNIST | 1,000 | 0.9266 | 0.8937 |
| Fashion-MNIST | 500 | 0.9206 | 0.8851 |
| CIFAR-10 | 1,000 | 0.8053 | 0.6077 |
| CIFAR-10 | 500 | 0.7089 | 0.5420 |
| CIFAR-100 | 1,000 | 0.9808 | 0.9499 |
| CIFAR-100 | 500 | 0.9793 | 0.9182 |

The phase structure held up qualitatively across all datasets: all correlations were positive and ranged from 0.54 to 0.99. Its strength varied with dataset difficulty rather than simply with dataset size. MNIST and, surprisingly, CIFAR-100 had near-perfect correlations, even though CIFAR-100 supplies far fewer images per class. CIFAR-10 was the clear outlier, with the weakest values (0.81/0.61 at 1,000 training examples, falling to 0.71/0.54 at 500) despite having ten times more images per class than CIFAR-100. Reducing the training set from 1,000 to 500 mildly weakened correlations overall; the largest changes were for ReLU on CIFAR-10 and CIFAR-100. This pattern suggests that lower training counts can compound the effect of dataset difficulty. In this setup, NNGP uncertainty calibration appears more sensitive to intrinsic visual complexity than to the number of training examples or images per class.

## 5. Known Limitations / Deviations

- Only one hyperparameter setting (`depth=3`, `weight_var=2.0`, `bias_var=0.2`) was evaluated rather than a full grid sweep, due to compute and time constraints.
- CIFAR-100 with 500 training examples provides only five training images per class. This is a very thin per-class sample and limits how broadly that result can be interpreted.
- Identical mean-subtraction preprocessing was used across datasets, although it may not be optimal for every dataset.
- The project uses the pinned TensorFlow 1.15 Docker image, consistent with the original repository’s environment.
