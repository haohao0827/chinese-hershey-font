# 中文单线字体（Chinese Hershey Font）
*使用计算机视觉将汉字转换为单线字体*

- **查看[在线演示](https://lingdong-.github.io/chinese-hershey-font/)！**
- **下载Hershey字体：[Heiti.hf.txt](dist/hershey/Heiti.hf.txt) (20975个Unicode字符), [Mingti.hf.txt](dist/hershey/Mingti.hf.txt) (73874个Unicode字符)**

像[Hershey字体](https://en.wikipedia.org/wiki/Hershey_fonts)这样的单线字体在制作酷炫的程序化图形和雕刻方面非常有用。对于神经网络来说，它们也更容易学习。

这个工具可以根据常规的True Type字体文件（ttf/ttc）自动生成中文单线字体。它可以输出经典的Hershey格式，也可以输出包含所有多段线的JSON文件。

该算法通过从多个不同角度扫描字符的光栅渲染图像，找到最可能构成笔画的线段。然后通过连接、合并和清理这些线段来估计笔画。

![](doc/screen002.png)

## 依赖项

- Python 3
- PIL/Pillow (`pip install pillow`)

## 文件格式

本软件可以生成以下类型的文件来编码单线字体。

### 笔画文件（Stroke Files）

一个JSON文件，包含一个对象。对象的键是字符的Unicode索引。每个键映射到一个多段线数组。多段线是一个点的数组。点是一个包含x和y坐标的2元素数组。坐标是0.0到1.0之间的浮点数，(0,0)是左上角。例如：

```json
{
  "U+4E00":[[[0.0, 0.55], [1.0, 0.55]]],
  "U+4E01":[[[0.02, 0.02], [0.99, 0.02]], [[0.51, 0.02], [0.53, 0.925], [0.31, 1.0]]]
}
```
上面的编码包含了Unicode中的前两个中文字符"一"和"丁"。

### Hershey字体

> Hershey字体是由Allen Vincent Hershey博士于1967年左右在海军武器实验室开发的一套矢量字体集合，最初设计用于在早期阴极射线管显示器上使用矢量进行渲染。这些字体公开可用且使用限制很少。矢量字体可以轻松地在二维或三维空间中缩放和旋转；因此，Hershey字体已广泛应用于计算机图形学、计算机辅助设计程序，以及最近在计算机辅助制造应用（如激光雕刻）中。（[维基百科](https://en.wikipedia.org/wiki/Hershey_fonts)）

[此链接](http://paulbourke.net/dataformats/hershey/)概述了如何解析Hershey字体。您也可以在[Lingdong-/p5-hershey-js](https://github.com/LingDong-/p5-hershey-js)找到我自己的实现。

与笔画文件相比，Hershey字体体积小很多倍，但坐标精度较低。

## 使用方法

### 预编译字体

如果您只想使用预生成的单线字体，可以在以下位置获取：

- Hershey字体可以在[dist/hershey](dist/hershey)文件夹中找到。
  - [Heiti.hf.txt](dist/hershey/Heiti.hf.txt)：**20975**个字符，基于[思源黑體(Souce Han Sans)](https://github.com/adobe-fonts/source-han-sans)。包括所有Unicode基本CJK表意文字。
  - [Kaiti.hf.txt](dist/hershey/Kaiti.hf.txt)：**20975**个字符，基于[全字庫正楷體(TW-Kai)](https://data.gov.tw/dataset/5961)。
  - [Mingti.hf.txt](dist/hershey/Mingti.hf.txt)：**73874**个字符，基于[花園明朝體(Hanazono)](https://zh.wikipedia.org/wiki/花園字體)。包括所有Unicode基本CJK表意文字以及CJK表意文字扩展A-E。
  - [Heiti-small.hf.txt](dist/hershey/Heiti-small.hf.txt)：**20975**个字符，与Heiti.hf.txt类似，但字体大小和间距较小。
  - [Mingti-basic.hf.txt](dist/hershey/Mingti-basic.hf.txt)：**27631**个字符，是Mingti.tf.txt的子集，包含基本CJK表意文字和扩展A。
- 笔画文件（包含多段线坐标的JSON）可以在[dist/json](dist/json)文件夹中找到。

**注意：**

`Heiti.hf.txt`和`Kaiti.hf.txt`最初基于macOS系统专有字体`STHeiti`和`STKaiti`。现已被替换为开源替代品。

### 生成笔画文件

如果您想从自定义TTF/TTC文件生成新的单线字体，首先使用以下命令生成JSON编码的笔画文件。

```
python char2stroke.py build 路径/到/字体.ttf
```
可选参数：

```
  --first [FIRST]
  --height [HEIGHT]
  --last [LAST]
  --ngradient [NGRADIENT]
  --output [OUTPUT]
  --strw [STRW]
  --width [WIDTH]
```
- `width`和`height`决定要扫描的字符光栅图像的尺寸。这些数字越大，细节越多，速度越慢。默认值均为`100`。
- `strw`是近似笔画宽度（以像素为单位，在给定的`width`和`height`下）。在合并笔画时使用。默认值为`10`。
- `first`和`last`指定要包含的Unicode字符范围。默认值为`0x4e00`和`0x9fef`，包含所有"CJK表意文字"字形。
- `ngradient`是用于扫描图像的不同梯度数量。当`ngradient = 1`时，仅扫描0°、45°和90°的笔画，而在`2`、`3`和`4`时，还包括`atan(1/2)`、`atan(1/3)`和`atan(1/4)`的斜率。默认值为`2`。
- `output`：输出文件的写入路径。未指定时，程序写入`stdout`，可以使用`>`([信息](https://en.wikipedia.org/wiki/Redirection_(computing)))和`|`([信息](https://en.wikipedia.org/wiki/Pipeline_(Unix)))来重定向输出。

### 快速测试

在生成大文件之前，对小字符集进行快速测试、比较不同字体的效果并调整参数是很有帮助的。以下命令通过并排显示计算机视觉结果来促进这一过程。

```
python char2stroke.py test 路径/到/字体1.ttf 路径/到/字体2.ttf ...
```
可选参数：

```
  --corpus [CORPUS]
  --height [HEIGHT]
  --ngradient [NGRADIENT]
  --nsample [NSAMPLE]
  --strw [STRW]
  --width [WIDTH]
```
- `corpus`：要测试的字符串。默认是[千字文](https://en.wikipedia.org/wiki/Thousand_Character_Classic)的文本。
- `nsample`：从语料库中随机选取的字符数。默认值为`8`。
- 其他参数与`build`模式相同。

### 从笔画文件生成Hershey字体

以下命令根据上一步生成的笔画文件生成Hershey字体。

```
python tohershey.py 路径/到/输入.json > 路径/到/输出.hf.txt
```

## 示例

![](doc/screen001.png)