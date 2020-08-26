---
layout: post
title: "linux笔记: cuda8+pytorch 使用gpu计算"
subtitle: "pytorch安装和简单测试案例"
author: Xiaolei.liang
header-img: "img/post-bg-unix-linux.jpg"
catalog: true
tags:
  - linux

---
## 先决条件
使用官方镜像，已经安装了 nvidia 驱动 和 cuda。
1. 查看cuda版本： `cat /usr/local/cuda/version.txt`
2. 查看nvidia 驱动信息:  `nvidia-smi`

## 1. 实例

python 实例如下：

```
# -*- coding: utf-8 -*-

import torch
import torchvision
import torchvision.transforms as transforms

transform = transforms.Compose(
    [transforms.ToTensor(),
     transforms.Normalize((0.5, 0.5, 0.5), (0.5, 0.5, 0.5))])

trainset = torchvision.datasets.CIFAR10(root='./data', train=True,
                                        download=True, transform=transform)
trainloader = torch.utils.data.DataLoader(trainset, batch_size=4,
                                          shuffle=True, num_workers=2)

testset = torchvision.datasets.CIFAR10(root='./data', train=False,
                                       download=True, transform=transform)
testloader = torch.utils.data.DataLoader(testset, batch_size=4,
                                         shuffle=False, num_workers=2)

classes = ('plane', 'car', 'bird', 'cat',
           'deer', 'dog', 'frog', 'horse', 'ship', 'truck')

from torch.autograd import Variable
import torch.nn as nn
import torch.nn.functional as F


class Net(nn.Module):
    def __init__(self):
        super(Net, self).__init__()
        self.conv1 = nn.Conv2d(3, 6, 5)
        self.pool = nn.MaxPool2d(2, 2)
        self.conv2 = nn.Conv2d(6, 16, 5)
        self.fc1 = nn.Linear(16 * 5 * 5, 120)
        self.fc2 = nn.Linear(120, 84)
        self.fc3 = nn.Linear(84, 10)

    def forward(self, x):
        x = self.pool(F.relu(self.conv1(x)))
        x = self.pool(F.relu(self.conv2(x)))
        x = x.view(-1, 16 * 5 * 5)
        x = F.relu(self.fc1(x))
        x = F.relu(self.fc2(x))
        x = self.fc3(x)
        return x


net = Net()
net.cuda()

import torch.optim as optim

criterion = nn.CrossEntropyLoss()
optimizer = optim.SGD(net.parameters(), lr=0.001, momentum=0.9)

for epoch in range(2):  # 循环遍历数据集多次

    running_loss = 0.0
    for i, data in enumerate(trainloader, 0):
        # 得到输入数据
        inputs, labels = data

        # 包装数据
        inputs, labels = Variable(inputs.cuda()), Variable(labels.cuda())

        # 梯度清零
        optimizer.zero_grad()

        # forward + backward + optimize
        outputs = net(inputs)
        loss = criterion(outputs, labels)
        loss.backward()
        optimizer.step()

        # 打印信息
        # print(len(loss.data))
        running_loss += loss
        if i % 2000 == 1999:    # 每2000个小批量打印一次
            print('[%d, %5d] loss: %.3f' %
                  (epoch + 1, i + 1, running_loss / 2000))
            running_loss = 0.0

        print('Finished Training')

```



> 使用cpu计算：
>
> 1. 去除 net.cuda() ；
>
> 2. inputs, labels = Variable(inputs.cuda()), Variable(labels.cuda()) 修改为: inputs, labels = Variable(inputs), Variable(labels)



## 2. 安装和测试

1. 安装pytorch

   ```
   pip3 install torch==1.0.0 torchvision==0.2.1 Pillow==6.2.2
   ```

2. 测试运行

   ```
   python3 test.py
   ```



## 3. 参考

* [https://pytorch.apachecn.org/docs/0.3/blitz_cifar10_tutorial.html](https://pytorch.apachecn.org/docs/0.3/blitz_cifar10_tutorial.html)

