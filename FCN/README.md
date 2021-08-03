# **"Fully Convolutional Network for Semantic Segmentation"를 참고하였습니다.**
---
# **이론**
## ***Fully convolutional network**

LeNet, AlexNet과 같은 CNN은 고정된 사이즈의 인풋을 가지고, fully connected layer이 고정된 차원을 갖고 공간좌표를 버리기 때문에 공간 정보가 없는 아웃풋을 갖습니다. 따라서 CNN은 Semantic Segmentation에 사용할 수 없습니다.

Semantic Segmentation을 하기 위해선 공간 정보가 필요합니다. 따라서 CNN의 fully connected layer을 fully convolution layer로 바꾸면 공간 정보를 보존하여 classification net이 heatmap을 생성할 수 있게 됩니다.

conv + pooling을 하게 되면 크기가 감소하는 downsampling이 되는데 이를 input image의 크기로 복원하기 위한 방법으로 본 논문에서는 shift-and-stich 방법보다 skip connection을 사용한 upsampling 방법이 더 효과적이므로 shift-and-stich 방법은 사용하지 않습니다.
upsampling을 하기 위해 bilinear interpolation을 사용합니다. 

또한 본 논문에서는 전체 이미지에서 crop된 부분을 사용하는 patchwise training 보다 전체 이미지를 사용하는 fully convolutional training이 더 효과적이므로 전체 이미지를 사용하여 training합니다.

## ***Segmentation Architecture** 

classifier를 FCN으로 변경하고, upsampling과 pixelwise loss로 dense prediction을 하기 위해 이를 증가시킵니다. 그 다음, coarse, semantic, local, appearance 정보를 결합하기 위해 skip architecture를 추가합니다.

backbone으로 VGG 16-layer net을 사용합니다. 그 다음 classifier layer인 fully connected layer를 버리고 fully convolutional layer로 변경합니다. 그리고 21차원 채널의 1x1 convolution을 추가하여 coarse output location에서 배경을 포함하는 PASCAL class를 예측하고 input image의 크기와 맞춰주기 위해 upsampling을 수행하는 deconvolution layer가 뒤따릅니다.

FCN-32s를 보면 픽셀들이 뭉쳐져 있습니다. 이를 해결하기 위해 fine prediction layer과 lower layer를 결합합니다. 이것이 skip layer입니다.
pool 4 위에 1x1 convolutional layer를 추가하고 stride 32인 conv7(fc7)에서 계산된 예측 값을 2x upsampling 한 뒤 더합니다. 그 다음 input image의 크기로 upsampling 하면 FCN-16s가 됩니다.
다시 이것을 2x upsampling 한 뒤 pool3의 값과 더하고 input image의 크기로 upsampling 하면 FCN-8s가 됩니다. FCN-32s, FCN-16s, FCN-8s 중 FCN-8s가 가장 정확합니다.

SGD with momentum을 하용하여 train하고, minibatch size of 20 image를 사용하며 learning rate는 10^-4  을 사용합니다. 또한 momentum을 0.9, 5^-4 또는 2^-4 의 weight decay, 두배의 learning rate for bias를 사용합니다. Dropout은 원래 classifier net에 포함되어 있습니다.
