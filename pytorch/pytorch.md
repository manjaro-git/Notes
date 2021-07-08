# Pytorch

## 1. Tensors

在pytorch中我们使用tensor 来编码模型的参数，输入和输出。

**1.1  基本知识** 

**与numpy ndarray的区别**

+ tensor 可以放在GPU或者其他的加速器上

+ tensor可以自动求导

+ tensor 和 numpy ndarray 也会共享内存，这样就不必拷贝数据了

  

**1.2 tensor的初始化**

	1. 直接从数据初始化 torch.tensor(data)
	2. 从numpy 转换   torch.from_numpy(n_array)
	3. 从另一个tensor转换  torch.ones_like(x_data) # retains the properties of x_data
		example2:
			x_rand = torch.rand_like(x_data, dtype = torch.float) #这里dtype必须加上，因为x_rand是long类型，我们使用random是float类型，XXX_like()函数会保存原数据的各种属性
	4. 使用随机值或者常量值 
		例子 : shape = (2, 3, ) # shape 是表示形状的元组
			rand_tensor = torch.rand(shape)


**1.3 tensor的属性**

   	1. shape
   	2. datatype
   	3. device


~~~python
print(f"shape of tensor:{tensor.shape}")

print(f"datatype of tensor: {tensor.dtype}")

print(f"device of tensor :{tensor.device}")

~~~



**1.4 operation of tensor**

tensor 的操作总共有一百多种，包括数值计算，线性代数，矩阵操作，采样等,

我们的tensor默认在cpu上，如果我们想把数据放在GPU上，就必须显式地使用.to()函数，当然需要先检查是否可以使用GPU，如下:

~~~python
# move tensor to gpu if avaliable
if torch.cuda.is_available():
    tensor = tensor.to("cuda")
~~~

**1）. 标准的类numpy 的索引和切皮**

~~~python
tensor = torch.ones(4,4)
print('First row:', tensor[0])
print('Second column:', tensor[:,1])
print('Last column:',tensor[:,-1])
tensor[:,1] = 0
print(tensor)
~~~

**2）.  拼接tensor**

~~~python
t1 = torch.cat([tensor, tensor, tensor],dim = 1) # 按照维度1拼接tensor
~~~

**3）.  数值操作**

~~~python
y1 = tensor @ tensor.T   # 矩阵乘法
y2 = tensor.matmul(tensor.T)

y3 = torch.rand_like(y2)
torch.matmul(tensor, tensor.T, out=y3)

# y1, y2, y3 结果一样

# element-wise product 
z1 = tensor * tensor
z2 = tensor.mul(tensor)
z3 = torch.rand_like(z2)
torch.mul(tensor,tensor,out = z3)
~~~



**4）. 单元素 tensor**

~~~python
agg = tensor.sum()
agg_item = agg.item()
print(agg_item, type(agg_item))
~~~

**5）. 置位 操作**

将运算结果存储在操作数中的操作，叫做置位(in-place)操作.它们都被表示为_,例如

~~~python
print(tensor,"\n")
tensor.add_(5)
print(tensor)
~~~

Notes:In-place operations save some memory, but can be problematic when computing derivatives because of an immediate loss of history. Hence, their use is discouraged.



**1.5 tensor 和 numpy 之间的转化**

tensor to numpyarray

~~~python
t = torch.onesy()
print(f"t:{t}")

n = t.numpy()
print(f"n:{n}")
~~~



tensor的改变也会反映在numpy arrays上面

~~~python
t.add_(1)
print(f"t: {t}")
print(f"n: {n}")
~~~

numpy arrays to tensor

~~~python
n = np.ones(5)
t = torch.from_numpy(n)
~~~

numpy 的改变也会反映在tensor上面

~~~python
np.add(n, 1, out = n)
print(f"t: {t}")
print(f"n: {n}")
~~~



## 2. DataSets & DataLoader

解释:

Code for processing data samples can get messy and hard to maintain; we ideally want our dataset code to be decoupled from our model training code for better readability and modularity. PyTorch provides two data primitives: `torch.utils.data.DataLoader` and `torch.utils.data.Dataset` that allow you to use pre-loaded datasets as well as your own data. `Dataset` stores the samples and their corresponding labels, and `DataLoader` wraps an iterable around the `Dataset` to enable easy access to the samples.

PyTorch domain libraries provide a number of pre-loaded datasets (such as FashionMNIST) that subclass `torch.utils.data.Dataset` and implement functions specific to the particular data. 

**Loading a Dataset**

load the FashionMNIST Dataset with following parameters:

+ **root** 是训练和测试数据存储的路径
+ **train** 指定是训练集还是测试集
+ **download = True** 如果在根目录找不到数据，就从网络上下载
+ **transform** and **target_transform** 指定特征和标签的转换

**下载**

~~~python
import torch
from torch.utils.data import Dataset
from torchvision import datasets
from torchvision.transforms import ToTensor
import matplotlib.pyplot as plt

train_data = datasets.FashionMNIST(root = "data", train=True, download= True, transform = ToTensor())

test_data = datasets.FashionMNIST(root = "data", train = False, download =True, transform = ToTensor())
~~~



**遍历和可视化**

想列表一样遍历Datasets ，使用matplotlib进行可视化

~~~python
labels_map = {  # 标签的映射
    0: "T-Shirt",
    1: "Trouser",
    2: "Pullover",
    3: "Dress",
    4: "Coat",
    5: "Sandal",
    6: "Shirt",
    7: "Sneaker",
    8: "Bag",
    9: "Ankle Boot",
}
figure = plt.figure(figsize=(8, 8))
cols, rows = 3, 3
for i in range(1, cols * rows + 1):
    sample_idx = torch.randint(len(training_data), size=(1,)).item()
    img, label = training_data[sample_idx]  # 像list一样访问数据集
    figure.add_subplot(rows, cols, i)
    plt.title(labels_map[label])
    plt.axis("off")
    plt.imshow(img.squeeze(), cmap="gray")  # 需要把tensor转换成可输出的信息
plt.show()
~~~



**创建自己的数据集**

要创建自己的数据集，就必须继承Dataset ,并且实现

~~~python
__init__ , __len__ , __getitem__
~~~

三个函数

具体的实现如下:

~~~python
import os
import pandas as pd
from torchvision.io import read_image

class CustomImgaeDataset(Dataset):
    def __init__(self,annotation_file, img_dir, transform =None, target_transform =None):
        self.img_labels = pd.read_csv(annotation_file)
        self.img_dir = img_dir
        self.transform = transform
        self.target_transform = target_transform

    def __len__(self):
        return len(self.img_labels)

    def __getitem__(self,idx):
        img_path = os.path.join(self.img_dir, self.img_labels.iloc[idx, 0])
        image = read_image(img_path)
        label = self.img_labels.iloc[idx,1]
        if self.transform:
            image = self.transform(image)
        if self.target_transform:
            label = self.tartget_transform(label)
        return image,label
~~~



**用DataLoader为模型训练准备数据**

The Dataset 一次取出一对 的特征和标签。当训练一个模型时，<1>我们以minibatch的形式传递samples，<2> 每一轮次都随机打乱数据，来减少过拟合, <3> 使用python的multiprocessing 来加速数据取出过程

DataLoader (迭代器)

~~~python
from torch.utils.data import DataLoader
train_loader = DataLoader(train_data, batch_size = 64, shuffle = True)
test_loader = DataLoader(test_data, batch_size = 64, shuffle = True)

~~~



**迭代整个DataLoader**

~~~python
train_features , train_labels = next(iter(train_loader))
print(f"Feature batch shape: {train_features.size()}")
print(f"Labels batch shape: {train_labels.size()}")

img = train_features[0].squeeze();
label = train_labels[0]
plt.imshow(img, cmap="gray")
plt.show()
print(f"Label:{label}")
~~~

