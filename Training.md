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
out vec2 TexColor;

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

















































