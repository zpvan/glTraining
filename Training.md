## 渲染一个三角形

### 定义顶点

```c
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
    0.5f, -0.5f, 0.0f,
    0.0f, 0.5f, 0.0f
}
```

利用顶点缓存对象(Vertex Buffer Object)来存储顶点数据, 它存在GPU显存中.

申请VBO

```c
unsigned int VBO;
glGenBuffers(1, &VBO); // 生成唯一ID
```

拿到ID, 但是不知道这个ID表示什么类型的数据, 指明这个是顶点缓存对象

```c
glBindBuffer(GL_ARRAY_BUFFER, VBO); // 指定为顶点缓存对象
```

将顶点数据复制到VBO

```c
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
/**
 * 参数1: 我们的顶点数据需要拷贝到的地方 (之前我们绑定的VBO)
 * 参数2: 数组的大小
 * 参数3: 数组的地址
 * 参数4: 指定显卡要采用什么方式来管理我们的数据, GL_STATIC_DRAW表示这些数据不会经常改变
**/
```

### 顶点着色器

```glsl
#version 330 core
layout (location = 0) in vec3 aPos;

void main() {
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}

/**
 * 第一行: 指明了使用OpenGL的版本以及运行模式(版本号3.3, 核心模式)
 * 第二行: 指明了需要从上一个步骤中获取一个vec3类型的位置数据, 数据位置在输入数据的0偏移位置(类似于
 *        输入了一块数据, 我们要的数据在头部)
 * 第七行(main函数内部): 将顶点的位置直接赋值成输入的位置, gl_Position是一个内置的变量, 用来表示
 *        顶点位置的
**/
```

编译着色器

```c
// 创建着色器对象
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);

// 编译着色器
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```

### 顶点着色器

```glsl
#version 330 core
out vec4 FragColor;

void main() {
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
}
```

编译着色器

```c
// 创建片元着色器对象
unsigned int fragementShader;
fragementShader = glCreateShader(GL_FRAGMENT_SHADER);
glShaderSource(fragmentShader, 1, &fragmentShaderSource, NULL);
glCompileShader(fragmentShader);
```

### 着色器程序对象

```c
// 链接着色器
unsigned int shadeProgram;
shaderProgram = glCreateProgram();
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
```

### 使用着色器程序对象

```c
// 使用着色器对象
glUseProgram(shaderProgram);
```

### 清理着色器

将已经附加过的着色器都清除掉, 已经不再需要了

```c
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```

### 指明顶点属性

我们可以输入想要输入的任何属性格式, 定义的顶点格式如下:

<img src="./data/vertex-format.png" style="zoom:80%"/>

我们的起始顶点的偏移为0, 每个位置分量占用4字节的空间, 一个顶点占用12字节空间, 于是, 指定顶点格式

```c
// 指明顶点格式
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void *)0);
glEnableVertexAttribArray(0);
/**
 * 参数1: 指明我们想要配置的顶点属性. 类似编号的东西, 之前我们设置了location = 0, 
 *       就是我们在这里用到的0
 * 参数2: 顶点属性的大小. 我们用到的顶点是一个vec3的结果, 所以大小为3
 * 参数3: 数据的类型. 我们使用的是float类型
 * 参数4: 指明数据是否要被规范化. 这里我们设置成FALSE, 不用规范化, 因为我们已经规范化好了.
 * 参数5: 表示属性的跨度. 正如之前我们分析的, 我们的跨度是12, 就是3倍的float类型
 * 参数6: 指明了数据的起始偏移量. 这里转成了一个void *指针类型比较奇怪
**/
```

我们获取的顶点数据是由VBO决定的, 而glVertexAttribPointer操作的是当前绑定到GL_ARRAY_BUFFER上的VBO, 所以, 我们当前操作的就是我们之前生成并绑定的那个VBO.

glEnableVertexAttribArray(0)是用来让顶点属性生效的, 参数0就是我们之前配置的那个顶点属性的位置.

### 绘制三角形

```c
glDrawArrays(GL_TRIANGLES, 0, 3); // 0表示顶点数据的起始索引, 3表示有3个顶点
```

顶点数组对象(Vertex Array Object), 作用是来保存对顶点属性的调用. 这样, 当我们需要这些顶点属性的时候, 只需要简单地绑定VAO, 不需要再设置一遍顶点属性就可以进行绘制.(如果不想保存, 只想显示, 不用VAO, 是不能显示出图像的)

VAO会保存两种东西:

一是对glEnableVertexAttribArray或者是glDisableVertexAttribArray的调用

二是使用glVertexAttribPointer设置的顶点属性以及与顶点属性相关连的VBO

```c
// 这段代码添加到生成VBO的代码之后
unsigned int VAO;
glGenVertexArrays(1, &VAO);
glBindVertexArray(VAO);
```

<img src="./data/vertex-handle-flow.png" style="zoom:80%"/>

这是一张顶点处理的流程图, 我们所做的工作就是处理其中的一些阶段. 顶点着色和片元着色.

## 元素缓存对象

### 元素缓存对象(EBO: Element Buffer Object)

在数组中标识出位置, 使用位置来绘制图像, 表面了大量存储重复数据. 解决了冗余数据的问题.

使用EBO的方式和使用VBO的方式一样, 先生成一个唯一ID, 然后绑定到OpenGL, 再然后将定义好的索引数组保存到EBO中去, 最后进行绘制.

### 定义顶点数组和索引数组

4个顶点, 5个索引

```c
float rectVertices[] = {
    0.5f, 0.5f, 0.0f,    // 右上角
    0.5f, -0.5f, 0.0f,   // 右下角
    -0.5f, -0.5f, 0.0f,  // 左下角
    -0.5f, 0.5f, 0.0f    // 左上角
};

unsigned int indices[] = {
    0, 1, 3,  // 第一个三角形
    1, 2, 3   // 第二个三角形
};
```

### 获取唯一ID

```c
unsigned int EBO;
glGenBuffer(1, &EBO);
```

### 绑定到OpenGL

绑定VBO的参数是GL_ARRAY_BUFFER, 绑定EBO需要的是GL_ELEMENT_ARRAY_BUFFER

```c
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
```

### 复制索引数据到EBO中去

还是使用glBufferData, 不过复制的时候需要指明是EBO, 所以, 要用GL_ELEMENT_ARRAY_BUFFER参数

```c
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(indices), indices, GL_STATIC_DRAW);
```

### 绘制

绘制VBO使用glDrawArrays函数, 绘制EBO就要使用glDrawElements函数

```c
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, 0);
```

<img src="./data/VAO-VBO-EBO.png" style="zoom:80%"/>

## 着色器类

### 着色器代码的格式

```glsl
#version 版本号
in 数据类型 变量名;
out 数据类型 变量名;
uniform 数据类型 变量名;
void main() {
    // 处理过程
    输出变量 = 处理结果;
}
```

in表示从上一个阶段输入的数据, out表示这个阶段需要输出的数据, uniform表示全局的数据(应用里也能读取和写入这个变量, 这就是着色器和应用之间胡同数据的方法), 主函数main这能够包含了处理过程, 将处理结果赋值给输出变量

### 着色器之间的数据互通

```c
// 顶点着色代码
const char *vertexShaderSource = "#version 330 core\n"
    "layout (location = 0) in vec3 aPos;\n"
    "out vec3 ourColor;\n"
    "void main()\n"
    "{\n"
    "    gl_Position = vec4(aPos, 1.0);\n"
    "    ourColor = vec3(0.5f, 0.0f, 0.0f);\n"
    "}\0";

// 片元着色器代码
const char *fragmentShaderSource = "#version 330 core\n"
    "out vec4 FragColor;\n"
    "in vec3 ourColor;\n"
    "void main()\n"
    "{\n"
    "    FragColor = vec4(ourColor, 1.0f);\n"
    "}\0";
```

### 着色器和应用之间的数据互通

```c
// 片元着色器代码
const char *fragmentShaderSource = "#version 330 core\n"
    "out vec4 FragColor;\n"
    "uniform vec4 ourColor;\n"
    "void main()\n"
    "{\n"
    "    FragColor = ourColor;\n"
    "}\0";
```

要使用uniform, 需要两步:

- 获取该变量在着色器程序中的位置(这里是着色器程序, 不是片元着色器, 是顶点着色器和片元着色器链接进入的那个着色器程序)

- 通过glUniform4fv函数对其进行赋值

```c
float greenValue = 1.0f;
int ourColorLocation = glGetUniformLocation(shaderProgram, "ourColor");
glUseProgram(shaderProgram);
glUniform4f(ourColorLocation, 0.0f, greenValue, 0.0f, 1.0f);
```

## 纹理

### 什么是纹理

纹理, 英文是texture, 指的是一张二维的图片, 可以贴到物体上.

### 映射方式

以左下角为原点, 向右伸展到1.0的位置, 向上伸展到1.0的位置, 表示一整张的纹理图像.

<img src="./data/texture-cordinate.jpg" style="zoom:80%"/>

### 纹理环绕方式(Texture Wrapping)

通常, 纹理坐标的范围在(0, 0)到(1, 1)之间, 但是如果我们制定的坐标在这之外呢? OpenGL会重复绘制纹理图.

也提供了更多的选择方案:

* GL_REPEAT: 默认方案, 重复纹理图片
* GL_MIRRORED_REPEAT: 类似于默认方案, 不过每次重复的时候进行镜像重复
* GL_CLAMP_TP_EDGE: 将坐标限制在0到1之间, 超出的坐标会重复绘制边缘的像素, 变成一种扩展边缘的图案
* GL_CLAMP_TO_BORDER: 超出的坐标将会被绘制成用户指定的边界颜色

每种方案的显示效果截然不同

<img src="./data/texture-wrap.jpg" style="zoom:80%"/>

设置纹理环绕方式的方法是调用glTexParameteri函数, 具体方式如下:

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT); // 横坐标
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT); // 纵坐标
```

如果设定了GL_CLAMP_TO_BORDER的环绕方式, 想要指定边界颜色, 就需要使用glTexParameterfv函数

```c
// 在设置了环绕方式之后再调用
float borderColor[] = {1.0f, 1.0f, 0.0f, 1.0f}; // 指定成黄色
glTexparameterfv(GL_TEXTURE_2D, GL_TEXTURE_BOEDER_COLOR, borderColor);
```

### 纹理过滤(Texture Filtering)

纹理坐标采用了浮点数的形式, 表明了它和分辨率无关. OpenGL需要非常精确的计算出纹理像素(通畅被称为纹素)和纹理坐标之间的对应关系. 当你有一张低分辨率的纹理图, 但是需要用到一个非常大的物体上时, 这种操作的重要性就更加明显了. OpenGL提供了集中不同的方案来解决这个问题, 其中最重要的两种: GL_NEAREST和GL_LINEAR

* GL_NEAREST

  最近点过滤. 指的是纹理坐标最靠近哪个纹素, 就用哪个纹素. 这是OpenGL默认的过滤方式, 速度最快, 但是效果最差

* GL_LINEAR

  (双)线性过滤. 指的是纹理坐标位置附近的几个纹素值进行某种插值计算之后的结果. 这是应用最广泛的一种方式, 效果一般, 速度较快

效果图如下:

<img src="./data/texture-filtering.png" style="zoom:80%"/>

最近点过滤的像素痕迹非常明显, 而线性过滤的方式效果就好上很多, 虽然感觉很模糊, 但是我们完全能理解一张小图放大之后会模糊这件事

```c
// 设置过滤方式还是使用glTexParameteri
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_NEAREST); // 缩小时的过滤方式
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR); // 放大时的过滤方式
```

### Mipmaps

所谓的mipmaps, 就是一系列的纹理图片, 每一张纹理图的大小都是前一张的1/4, 直到剩最后一个像素为止. 看起来

<img src="./data/mipmaps-sample.png" style="zoom:80%"/>

当物体越离越远的时候, 就可以选用较小纹理去映射, 这样不仅效果好, 而且速度也快.

可以通过glGenerateMipmaps来弄这个图片.

对于刚好在两张图片之间的物体, 我们可以参考前面两种过滤方式, 最近点(采用最近的图)或者线性(采用两张图的加权平均). 这样, 我们就有了四种不同的过滤方案.

* GL_NEAREST_MIPMAP_NEAREST: 采用最近的mipmap图, 在纹理采样的时候使用最近点过滤采样
* GL_LINEAR_MIPMAP_NEAREST: 采用最近的mipmap图, 纹理采样的时候使用线性过滤采样
* GL_NEAREST_MIPMAP_LINEAR: 采用两张mipmap图的线性插值纹理图, 纹理采样的时候采用最近点过滤采样
* GL_LINEAR_MIPMAP_LINEAR: 采用两张mipmap图的线性插值纹理图, 纹理采样的时候采用线性过滤采样

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);  //都是线性过滤
// 注意: mipmaps是用来处理物体变小时如何进行贴图的问题, 所以需要设置GL_TEXTURE_MAG_FILTER成mipmap方式, 如果强行使用, 会报错
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

### 使用

#### 顶点数据

我们的顶点数组中需要三样东西: 位置, 颜色, 纹理坐标.

```c
float vertices[] = {
    // 位置               // 颜色           // 纹理坐标
     0.5f,   0.5f, 0.0f, 1.0f, 0.0f, 0.0f, 1.0f, 1.0f, // 右上角
     0.5f,  -0.5f, 0.0f, 0.0f, 1.0f, 0.0f, 1.0f, 0.0f, // 右下角
    -0.5f, -0.5f,  0.0f, 0.0f, 0.0f, 1.0f, 0.0f, 0.0f, // 左下角
    -0.5f,  0.5f,  0.0f, 1.0f, 1.0f, 0.0f, 0.0f, 1.0f // 左上角
}；
```

<img src="./data/vertex-array-struct.png" style="zoom:100%"/>

可以看到, 我们的跨度变成了32, 也就是8sizeof(float), 颜色的起始偏移值为3sizeof(float), 纹理的起始偏移值为6sizeof(float). 这样, 我们指定顶点属性的自然需要指定相应的位置和偏移.

颜色属性:

```c
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *)(3 * sizeof(float)));
glEnableVertexAttribArray(1);
```

纹理属性:

```c
glVertexAttribPointer(2, 2, GL_FLOAT, GL_FALSE, 8 * sizeof(float), (void *)(6 * sizeof(float)));
glEnableVertexAttribArray(2);
```

顶点着色器

```glsl
#version 330 core

layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aColor;
layout (location = 2) in vec2 aTexCoord;

out vec3 ourColor;
out vec2 TexCoord;

void main() {
    gl_Position = vec4(aPos, 1.0);
    ourColor = aColor;
    TexCoord = aTexCoord;
}
```

片元着色器

```glsl
#version 330 core

out vec4 FragColor;

in vec3 ourColor;
in vec2 TexCoord;

uniform sampler2D ourTexture;

void main() {
    FragColor = texture(ourTexture, TexCoord); // 对纹理指定位置进行采样
}
```

创建纹理

```c
unsigned int texture;
glGenTextures(1, &texture);

/**
 * 第一个参数是创建的纹理数量
 * 第二个参数是保存那么多数量的整型数组
**/
```

创建完之后, 我们需要绑定到OpenGL的环境里才能操作

```c
glBindTexture(GL_TEXTURE_2D, texture);
```

绑定完成后, 就把之前加载的图片数据放到纹理中去了

```c
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGB, width, height, 0, GL_RGB, GL_UNSIGNED_BYTE, data);
/**
 * 参数一: 指定目标纹理. GL_TEXTURE_2D就表示当前的操作会对绑定的2D纹理产生作用(GL_TEXTURE_1D和GL_TEXTURE_3D里边的东西就不会受影响)
 * 参数二: mipmap层级. 我们之后会调用glGenerateMipmap来创建, 这里只需要创建原始图就行了
 * 参数三: 我们需要保存的纹理格式. 我们的图片只有RGB信息, 所以用GL_RGB格式
 * 参数四和参数五: 纹理图片的宽高
 * 参数六: 一定要设置成0
 * 参数七和参数八: 源图片的格式和数据类型. 我们加载的图片中有RGB值, 并且以字节的方式保存
 * 参数九: 加载的图片数据
**/
glGenerateMipmap(GL_TEXTURE_2D);
```

## 显示不同的纹理

### 纹理单元

纹理是通过纹理单元这个东西绑定到OpenGL环境中的. 纹理单元应该是OpenGL中内置的关于纹理的一些配置, OpenGL会根据这些配置来操作纹理. 在OpenGL中, 纹理单元的数量至少有16个, 我们可以通过GL_TEXTURE0 … GL_TEXUTRE15来激活使用. 默认激活的是GL_TEXTURE0, 所以我们之前的操作都是针对GL_TEXTURE0的.

让我们来激活另一个纹理单元并且对它进行一些操作.

首先, 我们用glActiveTexture(GL_TEXTURE1)来激活纹理单元1, 使之可操作.

然后, 将一个新的纹理ID绑定到这个纹理对象上, 我们不妨将这个新ID定义成texture2, 调用glBindTexture(GL_TEXTURE_2D, texture2)进行绑定.

接下来, 如同之前一样, 设置好环绕和过滤方式.

```c
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
```

接着, 把图片加载进去, 绑定到当前的纹理单元上.

```c
glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, width, height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);
glGenerateMipmap(GL_TEXTURE_2D);
```

最后, 我们需要告诉OpenGL着色器采样器和纹理单元之间的对应关系.

同时, 片元着色器也需要做相应的改动:

```c
uniform sampler2D texture1;
uniform sampler2D texture2;

void main() {
    FragColor = mix(texture(texture1, TexCoord), texture(texture2, TexCoord), 0.2);
}
```

mix函数是对某个点的纹理进行混合运算, 0.2表示该点的颜色20%来自采样器2, 80%来自采样器1.

### 融合因子控制

直觉上的, 我们需要三步走:

* 第一步: 定义一个全局的融合因子
* 第二步: 点击上下箭头的时候设置这个值
* 第三步: 设置这个融合因子变量值

## 转换(数学基础知识)

### 向量

向量就是用来表示方向的. 向量包括大小和方向两个要素.

### 向量运算

#### 标量

$$
\begin{vmatrix} 
1\\
2\\
3\\
\end{vmatrix} + x = 
\begin{vmatrix} 
1 + x\\
2 + x\\
3 + x\\
\end{vmatrix}
$$

向量取反

向量取反



向量取反





#### 向量取反

$$
-
\begin{vmatrix} 
x\\
y\\
z\\
\end{vmatrix} = 
\begin{vmatrix} 
-x\\
-y\\
-z\\
\end{vmatrix}
$$

#### 向量加减

$$
\begin{vmatrix} 
1\\
2\\
3\\
\end{vmatrix}
+
\begin{vmatrix} 
4\\
5\\
6\\
\end{vmatrix}
= 
\begin{vmatrix} 
1 + 4\\
2 + 5\\
3 + 6\\
\end{vmatrix}
=
\begin{vmatrix} 
5\\
7\\
9\\
\end{vmatrix}
$$

#### 长度

#### 向量乘法

##### 点乘

##### 叉乘

### 矩阵

#### 比例变化

<img src="./data/matrix-scale.png" style="zoom:100%"/>

#### 平移

<img src="./data/matrix-translate.png" style="zoom:100%"/>

#### 旋转

##### 绕X轴旋转

<img src="./data/matrix-x-rotate.png" style="zoom:100%"/>

##### 绕y轴旋转

<img src="./data/matrix-y-rotate.png" style="zoom:100%"/>

##### 绕z轴旋转

<img src="./data/matrix-z-rotate.png" style="zoom:100%"/>

## 显示3D立方体

### 坐标系统

看看要显示一个3D物体是有多么复杂

#### 局部空间(物体空间)

电脑在工厂中制造的时候, 工人需要去考虑这颗螺丝需要装到电脑的什么位置, 这个就是局部空间. 在局部空间中, 物体位于空间的原点, 所有的调整就是基于物体的相对位置去调整的.

#### 世界空间

在OpenGL中, 我们使用模型矩阵讲物体放到世界空间中的某个位置上. 需要分成两个操作:

* 人眼的视线转换到正对-z轴的方向, 并将物体转换到以人眼为原点的位置上.
* 物体必须在人眼的视野之内

这就是观察空间和裁剪空间的概念.

#### 观察空间

观察空间是以摄像机的位置为原点, 观察方向为-z轴方向的一个空间. 我们通常会用一系列的平移和旋转变换来把世界空间中的物体转换到观察空间中. 用来执行这种变换的东西, 我们称为观察矩阵.

#### 裁剪空间

摄像机有朝向, 也有拍摄的视野范围, 所有在视野范围之外的东西都看不到, 都被剔除了.

在每个顶点着色器运行结束的时候, OpenGL希望所有的坐标都在一个指定的范围内, 所有超出范围的坐标都会被裁剪, 被抛弃, 剩下的坐标才会进入片元着色器阶段然后显示到屏幕上.

将物体从观察空间转换到裁剪空间的操作叫做投影变换(perspective projection), 用到的也就是一个投影变换的矩阵而已.

还是从现实世界中看到东西的角度去分析.

人眼在看东西时, 左右宽度上有一定的范围, 上下高度上也有一定的范围. 放到OpenGL中, 就是摄像机的左右和上下方向上都有一定的视野角度(FOV), 只有在这个角度范围内的东西, 才可能被看到.

<img src="./data/crop-space.png" style="zoom:80%"/>

从上面的图中可以看出, h和w就确定了摄像机上下左右可以看到的范围大小. 通常我们会设置上下左右的视野都是90度. 为了方便计算(这点非常重要! 想把所有的物体都渲染出来, 世界上所有的计算机的运算能力加起来都不够, 所有, 不要显示的就坚决剔除.), 我们也会设置一个近裁剪面和一个远裁剪面. 比近才见面更近的物体被剔除, 比远裁剪面更远的物体被剔除. 我们需要把两个裁剪面之间的所有物体都映射到投影平面上(投影平面可以在近裁剪面和远裁剪面之间, 图上只是一种情况), 其他的物体都被剔除!

#### 视口空间

视口空间, 可以简单地理解成应用窗口. 投影平面上的东西和窗口上的像素通过一一对应的方式映射到窗口, 在窗口上显示!

这一步由OpenGL完成, 我们不管.

### 旋转的3D盒子

定义所有顶点的工作非常复杂繁琐, 我这里直接把定义好的36个顶点都列出来(6个面 * 每个面2个三角形 * 每个三角形3个顶点), 每个面顶点都包含了纹理坐标.

```c
float vertices[] = {
    -0.5f, -0.5f, -0.5f, 0.0f, 0.0f,
    0.5f, -0.5f, -0.5f, 1.0f, 0.0f,
    0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
    0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
    -0.5f, 0.5f, -0.5f, 0.0f, 1.0f,
    -0.5f, -0.5f, -0.5f, 0.0f, 0.0f,

    -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,
    0.5f, -0.5f, 0.5f, 1.0f, 0.0f,
    0.5f, 0.5f, 0.5f, 1.0f, 1.0f,
    0.5f, 0.5f, 0.5f, 1.0f, 1.0f,
    -0.5f, 0.5f, 0.5f, 0.0f, 1.0f,
    -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,

    -0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
    -0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
    -0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
    -0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
    -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,
    -0.5f, 0.5f, 0.5f, 1.0f, 0.0f,

    0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
    0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
    0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
    0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
    0.5f, -0.5f, 0.5f, 0.0f, 0.0f,
    0.5f, 0.5f, 0.5f, 1.0f, 0.0f,

    -0.5f, -0.5f, -0.5f, 0.0f, 1.0f,
    0.5f, -0.5f, -0.5f, 1.0f, 1.0f,
    0.5f, -0.5f, 0.5f, 1.0f, 0.0f,
    0.5f, -0.5f, 0.5f, 1.0f, 0.0f,
    -0.5f, -0.5f, 0.5f, 0.0f, 0.0f,
    -0.5f, -0.5f, -0.5f, 0.0f, 1.0f,

    -0.5f, 0.5f, -0.5f, 0.0f, 1.0f,
    0.5f, 0.5f, -0.5f, 1.0f, 1.0f,
    0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
    0.5f, 0.5f, 0.5f, 1.0f, 0.0f,
    -0.5f, 0.5f, 0.5f, 0.0f, 0.0f,
    -0.5f, 0.5f, -0.5f, 0.0f, 1.0f
};
```

修改顶点属性的设置

```c
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void *)0);
glEnableVertexAttribArray(0);
glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, 5 * sizeof(float), (void *)(3 * sizeof(float)));
GLES30.glEnableVertexAttribArray(1);
```

修改顶点着色器中的纹理坐标位置

```c
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec2 aTexCoord;
```

显示的方式

```c
glDrawArray(GL_TRIANGLES, 0, 36);
```

接下来就是走一个显示流水线, 局部空间->世界空间->观察空间->裁剪空间->视口空间

定义模型变换矩阵

```c
glm::mat4 model;
model = glm::rotate(model, (float)glfwGetTime() * glm::radians(50.0f), glm::vec3(0.5f, 1.0f, 0.0f));
```

模型会根据运行时间旋转一定的角度, 看起来有动画的效果.

定义观察变换矩阵

```c
glm::mat4 view;
view = glm::translate(view, glm::vec3(0.0f, 0.0f, -4.0f));
```

定义投影变换矩阵

```c
glm::mat4 projection;
projection = glm::perspective(glm::radians(45.0f), (float)SCR_WIDTH / (float)SCR_HEIGHT, 0.1f, 100.0f);
/**
 * 第一个参数: FOV, 角度为45度
 * 第二个参数: 定义了屏幕宽高比(aspect ratio), 这个值影响显示到窗口中的物体是原样显示还是被拉伸
 * 第三个参数: 0.1f是近裁剪面
 * 第四个参数: 100.0f是远裁剪面
**/
```

将这些矩阵应用到顶点着色器中, 顶点着色器需要3个变量来接收这些矩阵.

```glsl
uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main() {
    gl_Position = projection * view * model * vec4(aPos, 1.0f);
    TexCoord = vec2(aTexCoord.x, aTexCoord.y);
}
```

主循环中也要每次给这个三个变量赋值:

```c
shader.setMat4("model", glm::value_ptr(model));
shader.setMat4("view", glm::value_ptr(view));
shader.setMat4("projection", glm::value_ptr(projection));
```

在绘制三角形的时候, 是逐个绘制的. OpenGL没有检测它们在位置上的前后关系, 只是用后来的像素覆盖掉之前的像素而已.

我们需要开启深度检测!

OpenGL内部本就保存了一份顶点的深度信息, 这个信息的名字叫z缓存(z-buffer), 也叫深度缓存. 默认情况下, 深度检测是关闭的, 我们需要在某个位置把它打开. 打开的方法是调用

```c
glEnable(GL_DEPTH_TEST);
```

然后在清楚屏幕的时候, 也需要把深度缓存的数据清除掉

```c
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT);
```

如果要绘制10个, 我们不用傻傻地复制36个顶点9次, 只需要把这个盒子移动某个位置, 然后绘制一遍就行了.

定义10个位置:

```c
glm::vec3 cubePositions[] = {
    glm::vec3(0.0f, 0.0f, 0.0f),
    glm::vec3(2.0f, 5.0f, -15.0f),
    glm::vec3(-1.5f, -2.2f, -2.5f),
    glm::vec3(-3.8f, -2.0f, -12.3f),
    glm::vec3( 2.4f, -0.4f, -3.5f),  
    glm::vec3(-1.7f,  3.0f, -7.5f),  
    glm::vec3( 1.3f, -2.0f, -2.5f),  
    glm::vec3( 1.5f,  2.0f, -2.5f), 
    glm::vec3( 1.5f,  0.2f, -1.5f), 
    glm::vec3(-1.3f,  1.0f, -1.5f)
};
```

然后在主循环中绘制10次

```c
glBinderVertexArray(VAO);
for (int i = 0; i < 10; i++) {
    glm::mat4 model;
    model = glm::translate(model, cubePosition[i]);
    float angle = 20.0f * i;
    model = glm::rotate(model, glm::radians(angle), glm::vec3(1.0f, 0.3f, 0.5f));
    shader.setMat4("model", glm::value_ptr(model));
    glDrawArrays(GL_TRIANGLES, 0, 36);
}
```

<img src="./data/coordinate-line.png" style="zoom:100%"/>

## FPS摄像机

我们要创建一个类似FPS的摄像机, 它可以移动, 可以转头, 可以变焦(Kar 98K, 8倍镜的效果)

主要内容如下

* 观察空间变换的内部原理
* 键盘操纵摄像机前后左右移动的方法
* 鼠标操纵摄像机上下左右转动的方法
* 实现变焦的方式
* 将摄像机功能封装成一个很秀的类

### 观察(摄像机)空间

观察空间其实是以摄像机为原点, 以摄像机观察的方向为-z轴方向的坐标系统. 而观察矩阵的作用, 就是将场景中的物体从世界坐标转换到观察坐标. 要定义一个摄像机系统, 我们需要它在世界空间中的位置, 它的朝向, 以及一个向上方向的向量.

<img src="./data/view-coordinate-system.png" style="zoom:100%"/>

#### 相机位置

相机位置就是一个简单的向量, 表示其在世界空间中的位置.

```c
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 4.0f);
```

OpenGL是右手坐标系, 摄像机是往-z轴方向看的

#### 光线方向

作为朝向的反方向, 我成它为光线方向(物体反射光摄入观察者眼睛的方向). 计算的方式很简单, 将相机位置向量和观察目标点向量做减法就可以了. 我们使用世界坐标原点(默认点)作为我们的观察目标点.

```c
glm::vec3 cameraTarget glm::vec3(0.0f, 0.0f, 0.0f);
glm::vec3 cameraDirection = glm::mormalize(cameraPos - cameraTarget);
```

#### Right轴

我们下一个需要的向量是Right向量, 它表示坐标系统中的x轴正方向. 要计算这个Right向量, 我们要用到一个小技巧: 向量叉乘. Right向量必须要垂直于光线方向, 因此, 它必须要和光线方向与世界坐标系统的y轴组成的平面垂直. 根据叉乘规则, 我们只需要将y轴的单位向量与光线方向向量做叉乘就可以了.

```c
glm::vec3 up = glm::vec3(0.0f, 1.0f, 0.0f);
glm::vec3 cameraRight = glm::normalize(glm::cross(up, cameraDiretion));
```

#### Up轴

现在, 我们有了x轴和z轴, y轴已经呼之欲出了. 只需要用z轴向量叉乘x轴向量就可以了.

```c
glm::vec3 cameraUp = glm::cross(cameraDirection, cameraRight);
```

坐标系统的三个轴都有了, 马上开始生成观察矩阵

#### 观察矩阵

用矩阵的最大好处就是当你有了坐标空间的3个轴之后, 再加上一个位置向量就可以创造一个变换矩阵. 用这个矩阵乘上任何向量都可以将这个向量转换到观察坐标系中
$$
LookAt = 
\begin{vmatrix} 
\mathbf{R}_x & \mathbf{R}_y & \mathbf{R}_z & 0 \\
\mathbf{U}_x & \mathbf{U}_y & \mathbf{U}_z & 0 \\
\mathbf{D}_x & \mathbf{D}_y & \mathbf{D}_z & 0 \\
0 & 0 & 0 & 1 \\
\end{vmatrix}
*
\begin{vmatrix} 
1 & 0 & 0 & -\mathbf{P}_x \\
0 & 1 & 0 & -\mathbf{P}_y \\
0 & 0 & 1 & -\mathbf{P}_z \\
0 & 0 & 0 & 1 \\
\end{vmatrix}
$$
以上就是生成的观察矩阵.

R表示Right向量, U表示Up向量, D表示光线向量, P表示位置向量. 注意, 位置向量取的是它的反方向, 因为物体需要朝着摄像机相反的方向移动才行.

总结一下我们需要用到的数据: 摄像机的位置, 摄像机的观察目标(可以生成光线方向), 还有世界空间的Up向量. 使用这些数据, 通过计算, 我们就可以生成任意的观察矩阵. glm已经帮我们封装好了一个函数, 调用它, 我们可以直接获取到观察矩阵

```c
glm::mat4 view;
view = glm::lookAt(glm::vec3(0.0f, 0.0f, 4.0f),
                  glm::vec3(0.0f, 0.0f, 0.0f),
                  glm::vec3(0.0f, 1.0f, 0.0f));
```

 验证一下函数的效果. 我们把摄像机的位置放在半径为10的圆上, 让它的观察点始终在世界空间原点上, 并且, 摄像机不断地在圆上移动

```c
float radius = 10.0f;
float camX = sin(glfwGetTime()) * radius;
float camZ = cos(glfwGetTime()) * radius;
glm::mat4 view;
view = glm::lookAt(glm::vec3(camX, 0.0f, camZ),
                  glm::vec3(0.0f, 0.0f, 0.0f),
                  glm::vec3(0.0f, 1.0f, 0.0f));
```

### 移动相机

我们手动地控制相机的移动. 第一步, 我们要来创建一个相机系统, 这需要我们在程序开始的时候定义一些关于相机的变量

```c
glm::vec3 cameraPos = glm::vec3(0.0f, 0.0f, 4.0f);
glm::vec3 cameraFront = glm::vec3(0.0f, 0.0f, -1.0f);
glm::vec3 cameraUp = glm::vec3(0.0f, 1.0f, 0.0f);
```

观察矩阵就会变成这个样子:

```c
view = glm::lookAt(cameraPos, cameraPos + cameraFront, cameraUp);
```

我们希望摄像机的朝向不变而不是观察目标不变, 所以观察点就变成cameraPos + cameraFront. 现在观察点就变成cameraPos + cameraFront. 现在, 我们就要用键盘操作移动.

```c
float cameraSpeed = 0.05f; // 移动速度
if (glfwGetKey(window, GLFW_KEY_W) == GLFW_PRESS)
    cameraPos += cameraSpeed * cameraFront;

if (glfwGetKey(window, GLFW_KEY_S) == GLFW_PRESS)
    cameraPos -= cameraSpeed * cameraFront;

if (glfwGetKey(window, GLFW_KEY_A) == GLFW_PRESS)
    cameraPos -= glm::normalize(glm::cross(cameraFront, cameraUp) * cameraSpeed);

if (glfwGetKey(window, GLFW_KEY_D) == GLFW_PRESS)
    cameraPos += glm::normalize(glm::cross(cameraFront, cameraUp) * cameraSpeed);
```

这样就可以用WASD键来控制前后左右的移动了



这段代码纯粹是基于按键和代码运行速度来控制的, 如果机子不好, 代码运行慢点移动的速度也会变慢, 这就不太科学了. 因此, 我们引入时间来计算移动的距离.

先定义两个全局的变量, 用来保存上一帧绘制的时间以及两帧之间的间隔时间

```c
float deltaTime = 0.0f; // 两帧之间的间隔时间
float lastFrame = 0.0f; // 上一帧绘制的时间
```

然后, 每一帧都更新这两个数值

```c
float currentFrame = glfwGetTime();
deltaTime = currentFrame - lastFrame;
lastFrame = currenFrame;
```

最后, 使用这个数值

```c
float cameraSpeed = 2.5f * deltaTime; // 移动速度
```

但是, 左右方向上移动很快, 前后方向就很慢.

### 环顾四周

只用WASD的控制移动还不算一个完整的FPS摄像机, 我们还要能转头才行.

要实现转头的功能, 我们就要对cameraFront向量进行改变了. 不过对方向向量的改变比较复杂, 还涉及一些三角学的知识.

#### 欧拉角

欧拉角是绕着三条轴旋转的一个值. 一共有3种欧拉角, 分别是pitch, yaw和roll.

<img src="./data/euler-angles.png" style="zoom:100%"/>

pitch表示我们平时抬头低头的动作, yaw表示左看右看, roll表示打滚的效果. 每个欧拉角组合起来之后, 我们可以表示任何旋转.

作为一个FPS摄像机, 我们只需要pitch和yaw两种旋转就行了. 通过三角计算, 将方向向量设置成新值.

<img src="./data/pitch.png" style="zoom:80%"/>

上图就是pitch旋转的计算方法, 我们的初始方向为(0, 0, -1). 当我们想要转动pitch角度时, z坐标就等于-cos(pitch), y坐标就等于sin(pitch), 因为我们假定了斜边长度为1, 只考虑其方向.

<img src="./data/yaw.png" style="zoom:80%"/>

类似的, 计算yaw的方法也是如此, z坐标等于-cos(yaw),  x坐标等于-sin(yaw)

将两个旋转整合起来:

x = -sin(yaw) * cos(pitch)

y = sin(pitch)

z = -cos(pitch) * cos(yaw)

#### 鼠标输入

pitch和yaw的值是通过鼠标的移动得到的, 水平方向上的移动代表了yaw的值, 垂直方向上的移动代表了pitch的值. 我们需要保存上一次鼠标的位置, 这样可以通过计算和这次鼠标位置的差值算出转动的角度. 不过首先, 我们需要把鼠标的光标隐藏起来, 并且捕获鼠标消息

```c
glfwSetInputMode(window, GLFW_CURSOR, GLFW_CURSOR_DISABLED);
glfwSetCursorPosCallback(window, mouse_callback);
```

mouse_callback是响应鼠标消息的回调函数, 原型如下

```c
void mouse_callback(GLFWwindow *window, double xpos, double ypos);
```

window表示捕获的窗口, xpos表示x坐标, ypos表示y坐标

为了计算一个方向向量, 我们需要做这么几件事:

* 计算鼠标相对于上一次的位置偏移
* 将偏移值累加到摄像机的yaw和pitch值中去
* 添加一些旋转的限制
* 计算方向向量

```c
if (firstMouse) {  //设置初始位置，防止突然跳到某个方向上
    lastX = xPos;
    lastY = yPos;
    firstMouse = false;
}

float xoffset = lastX - xPos;   //别忘了，在窗口中，左边的坐标小于右边的坐标，而我们需要一个正的角度
float yoffset = lastY - yPos;   //同样，在窗口中，下面的坐标大于上面的坐标，而我们往上抬头的时候需要一个正的角度
lastX = xPos;
lastY = yPos;

float sensitivity = 0.05f;  //旋转精度
xoffset *= sensitivity;
yoffset *= sensitivity;

yaw += xoffset;
pitch += yoffset;

if (pitch > 89.0f)  //往上看不能超过90度
    pitch = 89.0f;
if (pitch < -89.0f)  //往下看也不能超过90度
    pitch = -89.0f;

glm::vec3 front;
front.x = -sin(glm::radians(yaw)) * cos(glm::radians(pitch));
front.y = sin(glm::radians(pitch));
front.z = -cos(glm::radians(pitch)) * cos(glm::radians(yaw));
cameraFront = glm::normalize(front);
```

为了防止突然跳到某个方向, 我们在鼠标刚开始的时候对它的位置进行设置. 接下来, 计算与上次位置的偏移量, 然后乘上旋转精度得到旋转的角度值. 然后, 将旋转角度累加到pitch和yaw值中去. 并且, 设置pitch的最大和最小值, 最后, 根据我们上面推导的公式, 计算方向向量, 并将其规范化.

#### 变焦

变焦功能, 就是狙击枪的放大镜头. 通过改变视野值来达到效果, 将fov值变小, 我们就能看到远方更精细的画面, 将fov值变大, 我们就可以看到更广的画面, 当然也失去了精度优势.

那么我们如何获得fov的改变值呢? 可以通过鼠标滚轮消息来模拟.

```c
//鼠标滚轮消息回调
void scroll_callback(GLFWwindow* window, double xoffset, double yoffset) {
    if (fov >= 1.0 && fov <= 45.0)
        fov -= yoffset;
    if (fov <= 1.0)
        fov = 1.0;
    if (fov >= 45.0)
        fov = 45.0;
}
```

当滚轮往前的时候, yoffset为正, 使得fov值变大, 物体变大变精细. 相反, 当滚轮往后的时候, yoffset为负, 使fov值变大, 物体变小视野变广.

修改投影矩阵

```c
projection = glm::perspective(glm::radians((float)fov), (float)SCR_WIDTH / (float)SCR_HEIGHT, 0.1f, 100.0f);
```

### 万向节死锁

使用四元数的方法, 解决这个问题























































