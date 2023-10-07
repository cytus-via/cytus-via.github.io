<div align='center' ><font size='70'>OpenGL</font></div>

# 前言
这篇笔记是本人跟随Learn OpenGL学习过程中记录的笔记和知识索引，结构也与Learn OpenGL大体相似，但同时也参考了OpenGL编程指南（原书第九版）的内容，因此结构上或许并不完全一直，作为我游戏引擎的第一篇笔记，生涩之处颇多，望见谅
<p align="right">2022年10月于北京</p>

# 目录
- [前言](#前言)
- [目录](#目录)
- [基本概念](#基本概念)
  - [OpenGL](#opengl)
  - [核心模式与立即渲染模式](#核心模式与立即渲染模式)
  - [glfw](#glfw)
    - [实例](#实例)
  - [glad](#glad)
    - [实例](#实例-1)
  - [Buffer](#buffer)
    - [顶点缓冲对象](#顶点缓冲对象)
    - [顶点数组对象](#顶点数组对象)
    - [元素缓冲对象](#元素缓冲对象)
  - [shader](#shader)
    - [vertex shader](#vertex-shader)
    - [tessellation shader](#tessellation-shader)
    - [geometry shader](#geometry-shader)
    - [fragment shader](#fragment-shader)
    - [glsl(OpenGL Shading Language)实例](#glslopengl-shading-language实例)
    - [着色器编译](#着色器编译)
---
# 基本概念
## OpenGL
OpenGL一般被认为是图形学的一个API，但是严格来说，OpenGL只是一个大家所共同制定的规范，或者说是一组规定好的接口，但对于各接口如何实现，各显卡厂商所编写的API库各不相同，开发者只针对接口开发，无论使用哪个库并不会有功能上的差异。
![library](./OpenGLLibrary.webp)
更进一步地，OpenGL可以被认为是一个巨大的状态机，开发者只需要对其中的一些状态进行设置，OpenGL就会控制显卡按照预设的渲染管线运行
## 核心模式与立即渲染模式
立即渲染模式封装了OpenGL大部分的功能，使得开发者可以更轻松地开发，但相应的，开发者无法控制太多的操作，灵活性也大大降低，从3.2版本开始，OpenGL使用了核心模式，使得开发者可以做更多事情，但相应的，对开发者的要求也更高
## glfw
glfw是OpenGL最常用的开源库之一（也可能没有之一），它为创建窗口、上下文和表面、接收输入和事件提供了一个简单的接口，而这些已经足够我们渲染出需要的东西了
### 实例
```CPP
//初始化一个glfw库
if (!glfwInit())
        return -1;
//创建一个窗口和上下文
GLFWwindow* window;
window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    if (!window)
    {
        glfwTerminate();
        return -1;
    }
glfwMakeContextCurrent(window);
//渲染过程的每帧循环
while (!glfwWindowShouldClose(window))
{
    //交换渲染过程中的前后帧缓存（稍后会提到）
    glfwSwapBuffers(window);
    //检查事件
    glfwPollEvents();
}
//关闭glfw库
glfwTerminate();
```
## glad
由于OpenGL对于不同的库、不同的平台可能有着不同的实现，因此对应的API地址也并不是固定的，需要在运行时加入大量的函数查找过程，这一步对于开发者而言是极为繁琐的，而glad库则帮我们处理了这一步骤
### 实例
```CPP
//加载glad库
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress))
    {
        std::cout << "Failed to initialize GLAD" << std::endl;
        return -1;
    }  
```
## Buffer
### 顶点缓冲对象
对于渲染的第一步，自然是设置我们要画出的三角形顶点数据，对于这些顶点数据，我们会将它存在显存中，并且通过一个顶点缓冲对象（VBO）来管理，它看起来会是这样的
```CPP
//顶点数据定义
float vertices[] = {
    -0.5f, -0.5f, 0.0f,
     0.5f, -0.5f, 0.0f,
     0.0f,  0.5f, 0.0f
};
//生成缓冲，VBO可以是数组，会在VBO中生成对应数量的缓冲
unsigned int VBO;
glGenBuffers(1, &VBO);
//将缓冲对象绑定到目标上，这会让OpenGL之后所有对GL_ARRAY_BUFFER的调用使用VBO作为缓冲对象
glBindBuffer(GL_ARRAY_BUFFER, VBO); 
//此处即将我们预先设置的顶点数据输入缓冲目标，由于前面已经绑定，数据会存储在VBO中，GL_STATIC_DRAW表明了数据会以什么方式使用，由于这里顶点数据不会随每一帧而更新，因此为STATIC
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
```
### 顶点数组对象
由于顶点属性在很多时候并没有固定的格式，OpenGL无法了解我们所设置的顶点数据有什么含义，因此，需要我们手动地解释顶点数据
```CPP
glVertexAttribPointer(0,                  //顶点属性所在位置
                      3,                  //每个顶点的属性大小
                      GL_FLOAT,           //顶点属性的变量类型
                      GL_FALSE,           //是否将数据标准化（归一化）
                      3 * sizeof(float),  //顶点数据步长
                      (void*)0            //位置数据在缓冲中起始位置的偏移量
);
```
当我们对多个物体进行绘制时，输入顶点数据并且指定解释方式将会一次次地调用，为了简化调用过程，引入了顶点数组对象（VAO）以记录VBO以及属性解释方式，它的使用看起来会像是这样：
```CPP
unsigned int VAO;
glGenVertexArrays(1, &VAO);
//绑定VAO
glBindVertexArray(VAO);
//将数据导入对应的VBO
glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(vertices), vertices, GL_STATIC_DRAW);
//设置顶点属性指针
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
//解绑VAO
glEnableVertexAttribArray(0);
```
绘制时只需绑定VAO即可
```CPP
glBindVertexArray(VAO);
```
### 元素缓冲对象
## shader
着色器用于将输入的顶点数据或其他数据进行处理以完成最终的渲染要求，现代OpenGL拥有诸多着色器，而其中大部分是可选的，但至少需要一个顶点着色器和一个片段着色器
### vertex shader
顶点着色器用于将每一个顶点数据（通常必然包含位置信息以及可能包含许多其他计算颜色可能需要的信息）进行处理，可能只是简单的将输入数据复制给下一阶段（此时叫传递着色器），也可能需要变换顶点的位置或者计算光照信息等
### tessellation shader
细分着色器是对顶点着色器的补充处理，包括将面片细分为更小的面片以得到更精细的效果或者将多个面片合并以节省开销等
### geometry shader
几何着色器会对几何图元做更进一步的处理，如创建新的图元
### fragment shader
在这一步中，我们会对之前的数据进行计算，将得到最终的颜色，这决定了最后看到的图像结果，这一阶段中包含了纹理贴图等诸多内容
### glsl(OpenGL Shading Language)实例
```CPP
//版本申明
#version 330 core
//申明一个输入属性
layout (location = 0) in vec3 aPos;

void main()
{
    gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);
}
```
```CPP
#version 330 core
//申明一个输出属性
out vec4 FragColor;

void main()
{
    FragColor = vec4(1.0f, 0.5f, 0.2f, 1.0f);
} 
```
### 着色器编译
```CPP
const char *vertexShaderSource = "#version 330 core\n"
    "layout (location = 0) in vec3 aPos;\n"
    "void main()\n"
    "{\n"
    "   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
    "}\0";
unsigned int vertexShader;
vertexShader = glCreateShader(GL_VERTEX_SHADER);
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);
```
通过一下方式将着色器链接为一个程序
```CPP
unsigned int shaderProgram;
shaderProgram = glCreateProgram();
glAttachShader(shaderProgram, vertexShader);
glAttachShader(shaderProgram, fragmentShader);
glLinkProgram(shaderProgram);
glUseProgram(shaderProgram);
glDeleteShader(vertexShader);
glDeleteShader(fragmentShader);
```