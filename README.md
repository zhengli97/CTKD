# Curriculum Temperature for Knowledge Distillation

> [**Curriculum Temperature for Knowledge Distillation**]() <br>
> Zheng Li, Xiang Li#, Lingfeng Yang, Borui Zhao, Renjie Song, Lei Luo, Jun Li, Jian Yang#. <br>
> Nankai University, Nanjing University of Science and Technology, Megvii Technology. <br>
> AAAI 2023 <br>
> [[Paper](https://arxiv.org/abs/2211.16231)]  [[Project Page](https://zhengli97.github.io/CTKD/)] [[中文解读](https://zhengli97.github.io/CTKD/chinese_interpertation.html)]

## 💡 Note 

The implementation of instance-wise temperature has been publicly released, please check the following **README** carefully.

## Abstract

CTKD organizes the distillation task from easy to hard through a dynamic and learnable temperature. 
The temperature is learned during the student’s training process with a reversed gradient that aims to maximize the distillation loss (i.e., increase the learning difficulty) between teacher and student in an adversarial manner.

As an easy-to-use plug-in technique, CTKD can be seamlessly integrated
into existing state-of-the-art knowledge distillation frameworks and brings general improvements at a negligible additional computation cost.

### Framework 

<div style="text-align:center"><img src="figure/framework.png" width="100%" ></div>

(a) We introduce a learnable temperature module that predicts a suitable temperature τ for distillation. The gradient reversal layer is proposed to reverse the gradient of the temperature module during the backpropagation. 

(b) Following the easy-to-hard curriculum, we gradually increase the parameter λ, leading to increased learning difficulty w.r.t. temperature for the student.

### Visualization

The learning curves of temperature during training:

<div style="text-align:center"><img src="figure/temp_curve.png" width="100%" ></div>


### Main Results

On CIFAR-100:

| Teacher <br> Student |RN-56 <br> RN-20|RN-110 <br> RN-32| RN-110 <br> RN-20| WRN-40-2 <br> WRN-16-2| WRN-40-2 <br> WRN-40-1 | VGG-13 <br> VGG-8|
|:---------------:|:-----------------:|:-----------------:|:-----------------:|:------------------:|:------------------:|:--------------------:|
| KD | 70.66 | 73.08 | 70.66 | 74.92 | 73.54 | 72.98 |
| **+CTKD** | **71.19** | **73.52** | **70.99** | **75.45** | **73.93** | **73.52** |

On ImageNet-2012:

|                 | Teacher <br> (RN-34) | Student <br> (RN-18) | KD | +CTKD | DKD | +CTKD |
|:---------------:|:---------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|:-----------------:|
| Top-1           | 73.96   | 70.26 | 70.83 | 71.32 | 71.13 | 71.51 |
| Top-5           | 91.58   | 89.50 | 90.31 | 90.27 | 90.31 | 90.47 |

## Requirements 

- Python 3.8
- Pytorch 1.11.0
- Torchvision 0.12.0

## Running

1. Download the pre-trained teacher models and put them to `./save/models`.

|  Dataset | Download |
|:---------------:|:-----------------:|
| CIFAR teacher models   | [[Baidu Cloud](https://pan.baidu.com/s/1ncvsfLTQ-GdXtKY-xtaweg?pwd=meaf)] [[Github Releases](https://github.com/zhengli97/CTKD/releases/tag/pretrained-models)]   |
| ImageNet teacher models  | [[Baidu Cloud](https://pan.baidu.com/s/1408PoziVAA8E3DojxUq1Hw?pwd=s4ma)] [[Github Releases](https://github.com/zhengli97/CTKD/releases/tag/pretrained-models)]   |

If you want to train your teacher model, please consider using `./scripts/run_cifar_vanilla.sh` or `./scripts/run_imagenet_vanilla.sh`.

After the training process, put your teacher model to `./save/models`.

2. Training on CIFAR-100:
- Download the dataset and change the path in `./dataset/cifar100.py line 27` to your current dataset path.
- Modify the script `scripts/run_cifar_distill.sh` according to your needs.
- Run the script.
    ```  bash
    sh scripts/run_cifar_distill.sh  
    ```

3. Training on ImageNet-2012:
- Download the dataset and change the path in `./dataset/imagenet.py line 21` to your current dataset path.
- Modify the script `scripts/run_imagenet_distill.sh` according to your needs.
- Run the script.
    ```  bash
    sh scripts/run_imagenet_distill.sh  
    ```

## Model Zoo & Training Logs

### Global Temperature

We provide complete training configs, logs, and models for your reference.

CIFAR-100:

- Combing CTKD with existing KD methods, including vanilla KD, PKT, SP, VID, CRD, SRRL, and DKD.  
(Teacher: RN-56, Student: RN-20)  
[[Baidu Cloud](https://pan.baidu.com/s/13-z-T4ooQDlWrm4isEH4qA?pwd=3bmy)] [[Google](https://drive.google.com/drive/folders/1pT8zmmOFMs5MqDLP6b4Cobv422CAcVF4?usp=sharing)]

### Instance-wise Temperature

The detailed implementation and training log of instance-wise CTKD are provided for your reference.  
[[Baidu Cloud](https://pan.baidu.com/s/1SG0dCLjTATIOOy2-2JFCbA?pwd=inx5)][[Google Drive](https://drive.google.com/drive/folders/12rgry4kXCmublAonPjTHfhhxE51zmF4Q?usp=sharing)]

In this case, you need to simply change the distillation loss calculation process in the `distiller_zoo/KD.py line16-line18` as follows:
```
KD_loss = 0
for i in range(T.shape[0]):
   KD_loss += KL_Loss(y_s[i], y_t[i], T[i])
KD_loss /= T.shape[0]
```
KL_Loss() is defined as follows:
```
def KL_Loss(output_batch, teacher_outputs, T):

    output_batch = output_batch.unsqueeze(0)
    teacher_outputs = teacher_outputs.unsqueeze(0)

    output_batch = F.log_softmax(output_batch / T, dim=1)
    teacher_outputs = F.softmax(teacher_outputs / T, dim=1) + 10 ** (-7)

    loss = T * T * torch.sum(torch.mul(teacher_outputs, torch.log(teacher_outputs) - output_batch))
    return loss
```

## Contact

If you have any questions, you can submit an issue on GitHub, leave a message on Zhihu Article (if you can speak Chinese), or contact me by email (zhengli97[at]qq.com).

## Citation

If this repo is helpful for your research, please consider citing our paper and giving this repo a star ⭐. Thank you!

```
@inproceedings{li2023curriculum,
  title={Curriculum temperature for knowledge distillation},
  author={Li, Zheng and Li, Xiang and Yang, Lingfeng and Zhao, Borui and Song, Renjie and Luo, Lei and Li, Jun and Yang, Jian},
  booktitle={Proceedings of the AAAI Conference on Artificial Intelligence},
  volume={37},
  number={2},
  pages={1504--1512},
  year={2023}
}
```
