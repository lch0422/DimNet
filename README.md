# DimNet

## 声明

`convNd.py` 文件代码来自 [pvjosue/pytorch_convNd](https://github.com/pvjosue/pytorch_convNd)

## 如果什么都不改，直接运行

直接运行 `train.py` 即可。

### 此时实际做了什么

`data` 文件夹下面按照 `000001.h5` ~ `00000x.h5` 的顺序命名存放的文件是数据集，每个 `.h5` 文件里面存放了一张 `5x5x80x80` 的待超分的光场图像和一张 `5x5x320x320` 的 HR 原图。

开始运行后，会先加载 `LF-DimNet_5x5_4xSR.pth.tar` 的预训练结果，然后开始训练，一旦出现损失函数更低的情况时，会立即把当前的模型参数保存到这个文件中（覆盖），也没有专门去设一个文件夹做训练结果更完整的整理了

## `train.py` 里面的参数

* `device` ：在什么设备上运行，填 `cpu` 、 `cuda:0` 等。

* `upscale_factor`：现在这个版本是把5x5x80x80的图像超分成5x5x320x320，所以填的是4，按理说只要matlab的图片预处理那边做对应的修改，这里也可以填其他的比例（不过还没试过）。

* `batch_size` ：设的是4，应该可以随便改，这个运行在大一些的显卡上面的话可能会跑得比较快，估计是以秒的数量级出一代，所以可以考虑把batchsize改大一点

* `lr` `n_epochs` `n_steps` `gamma`：其实关于训练这几个参数lr、epoch、step、gamma虽然都大概知道是什么意思但是还是不太知道调的时候的技巧，所以这几个也都是凭感觉经常瞎写一些数据上去（按理说应该还是多少有点规律的吧。。。感觉是个手艺活）

* `load_pretrain`：就是接着之前的训练，表示要加载之前的 `LF-DimNet_5x5_4xSR.pth.tar` 这个文件继续运行（前提是有这个文件，没有的话就只能设成 `False` 了）。文件里面其实只保存了模型网络里面的参数以及当时的损失函数，也没保存训练到第几代之类的。`False` 就会设目前最优值MSE是1，只要发现更小的损失函数就保存（所以如果原本放了个预训练参数在那里的话就会覆盖掉）。

* 其实最下面92行的地方还有一个 `num_works` ：没有改过，毕竟是在笔记本上，换大设备了应该也可以改。

注：现在这个网络只能处理5x5，因为这个网络的结构和子孔径数量有一定关系，我们其实还想了个更复杂的9x9的版本，从 drawio 的图里面也能看到，这个估计得后面写出来了再想办法改）。

## test

`test.py` ：运行以后应该是相当于会把 `data` 文件夹里面的每个图像都跑一遍，只是还没写求平均的，数据的汇总处理可能后面还要写一下。

`single_test.py` ：作用是跑单个的图片并且显示出来对比图，一个原图、一个低分辨率的图、一个超分的图。

## 其他

* 目前这个保存参数的写法，受图片影响很大，纯色图片的损失函数会很低，所以一旦刚好抽到了纯色图片一般就会记录这次的参数，但实际上这组参数并不一定好只是因为图片简单，增大batch size应该会缓解一下这个问题。
  之前那个InterNet网络里面好像用了求平均还是什么方法来弄，应该也比现在写的这个科学，估计后面还得改改，改得更合理一点

* 现在看到的 `LF-DimNet_5x5_4xSR.pth.tar` 是之前跑出来的数据，也就是个随便跑了一下的结果。
* 自己之前的训练方法其实就是一次跑100代，然后改点参数接着跑，所以其实一个数据到底是训练了多少代出来的最后也数不太清楚，这样主要的好处是在笔记本这种小设备上可以随时开始和结束，而且也比较方便随时调整，大型一点的机子估计就不太适合这么跑了，倒也不是不行，但可能得看看情况做点调整。
* 初始化用的0初始化。