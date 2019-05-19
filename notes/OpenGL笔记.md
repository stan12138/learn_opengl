## OpenGL笔记



我曾经学习过使用pyopengl写过opengl代码，但是遇到过很多的问题，其中给我带来最大问题的就是assimp，虽然现在我认为使用我的C/C++编译的知识，加上已经安装过的Visual studio我已经可以解决这个问题，但是我并不准备在去解决这个问题并继续这个项目。我决定尝试使用C++来继续，如果我某天决定要继续的话。在一切之前，我首先将会做一些与环境设置相关的工作，为可能的继续铺平道路。



### IDE与编译

是否要选择使用IDE完成opengl的编译？我选择否。于是我选择使用GNU的编译器手动完成编译和连接工作。



### 库与库的编译

我们需要opengl的库，还有重要的窗口管理库glfw,glad，以及后期可能需要的模型加载库assimp。

opengl的静态库已经存在于windows系统中了，我们不需要做特别的处理

glfw提供的是源码，当然也有提供windows上面的预编译文件，因为接下来我们要使用的是mingw中的GNU编译器，所以需要的是其中的mingw文件夹里面的静态链接库

关于如何使用，也可以说是如何编译opengl代码，我在`C与C++代码组织与编译`笔记中已经作为最后一部分做过记录，这里不再赘述。

关键是，在那里我提到了截止到目前为止，我还没有解决如何才能不使用预编译的glfw库，自行手动编译的问题，这里，我已经解决了。

### Cmake与编译

之前已经了解过了，makefile是用于编译的一个很好的工具，但是问题在于他不是跨平台的，这意味着作者可能需要为不同的平台各写一份适合这个平台的makefile或者类似makefile的文件，这很不合理。于是便有了cmake，cmake的存在可以让作者只需要按照cmake的语法写一份`CMakeLists.txt`的文件，然后就可以在不同的平台进行编译，具体来说就是使用不同平台的编译器，通过cmake可以分别生成适合于该平台的makefile或者类似makefile的文件，然后再通过make或类似make的操作完成最终的编译。

关于cmake的下载安装，就不多说了，完后可以选择使用GUI或者命令行，二者没有区别，我建议使用GUI。

分别选择glfw的源码的文件夹和结果要存放的文件夹，然后configer选择编译工具，如果要生成visual studio的文件，一般选择visual studio 14 2015，然后重新点击configer确认config，然后点击生成即可，生成结果即可用于visual studio

但是，我想要的是mingw，我之前的问题就在于此，一直无法使用mingw完成cmake，我选择了mingw makefile，却报错说未发现。直到今天我才找出错误在哪。这个过程实际上调用了mingw32-make.exe，还记得这个命令吧，就是我在c++编译中建议修改为make.exe的那个程序，就是因为我的修改导致cmake无法发现编译工具的，只要再重新提供一个mingw32-make.exe即可。

照和之前一样的步骤，生成完成之后，我们就可以在结果的文件夹中发现makefile文件和一堆其他文件，这意味着成功了，只需要继续执行make完成编译即可，在src文件夹中就可以找到`libglfw3dll.a`文件，这就是我们需要的链接库，虽然他不是我们之前一直使用的`libglfw3.a`但是我测试了一下也是可以的。

#### Assimp的编译

接下来我就尝试了一下assimp的编译，也作为测试cmake之用

同样的步骤，我下载了最新版的源码，同样选择mingw进行生成，会报几个错，但是无所谓了。

接下来，执行make，针对我所使用的版本(assimp 4.1.0)来说，在make的过程中会多次出现错误，但是都不至于停止，直到69%的时候，报错：

~~~
C:\Users\Stan\Downloads\assimp\assimp-4.1.0\tools\assimp_cmd\assimp_cmd.rc:4:10: fatal error: ../../revision.h: No such
file or directory
 #include "../../revision.h"
          ^~~~~~~~~~~~~~~~~~
compilation terminated.
~~~

然后make终止，无法进行，明显这个错误是未发现`revision.h`这个头文件，经过网上查找，按照报错的指示，在`assimp_cmd.rc`文件所处的文件夹上两层文件夹，新建`revision.h`头文件，内容为：

~~~
#ifndef ASSIMP_REVISION_H_INC
#define ASSIMP_REVISION_H_INC

#define GitVersion 0x0
#define GitBranch "master"

#endif // ASSIMP_REVISION_H_INC
~~~

保存，继续make即可，虽然还会报错，但不至于终止，直至100%结束

但是我检查发现生成的结果，发现make前后似乎区别并不大？我也不知道需要的究竟是哪些库，反正code文件夹中的两个静态链接库在make之前就有了，make之后也似乎并没有生成其他的.a库

并且，看网上的assimp使用示例，还需要很多头文件，这里也没有显式的给出整合的头文件夹，似乎还是需要手动从源码中搜集，总之虽然勉强算是make成功了，但是感觉不太对。

至于测试，assimp的使用似乎很复杂，所以我没有做测试，所以也不知道到底对不对。

learnopengl这个网站是我常用的opengl的参考网站，它上面关于assimp提到了它使用的版本是`assimp3.1.1`，并提到这个版本会存在需要DirectX的问题，以及DirectX的安装报错问题，是的很不幸，在我测试的过程中，他提到的问题我全都遇到了，然后，我解决到安装问题的时候就不想再继续了。

至于最新版本中就似乎不再需要DirectX了，所以也就不再有解决这个问题的必要了。

总之，现在就是这样。



### 代码组织

唉，又要继续代码组织问题了，手动编译确实会很烦。

我认为，规范的代码组织应该是这样的：

~~~
--headers
  -- xxx.h
  -- xxx.h
--lib
  --glad.c
  --glad.o
  --libglfw3dll.a
  --xxx.o
  --xxx.cpp
--images
  --xxx.png
--basic_texture.cpp
~~~

差不多吧，明白我的意思就好。

我们需要依赖的库和模块包括了:

- glfw的静态链接库
- glad.o
- stb_image.o
- Shader.o
- Texture.o

除此之外，系统提供的opengl,glu,gdi什么的就不说了

glfw的链接库是通过cmake得到的，所以我们不可能实时编译，肯定是预先弄好的

glad的情况其实略复杂，它会依赖一些特定路径的头文件，原则上，这对我来说应该完全不是问题，解决这个问题的方法多得是，可以通过按照他的期望存储文件，也可以通过修改源码，还可以通过在gcc命令中添加参数。但是我想规范的话的时候，却出了点问题，我也无心解决，所以，目前的暂缓之计就是按照他的期望放置，然后编译，再把得到的文件直接放进lib

至于stb_image库，这是一个用于导入图片的库，这个库的使用很简单，他只是一个单一的头文件而已，但是我们还是需要做一些额外的工作，新建一个cpp文件，可以任意命名，但是最好命名为`stb_image.cpp`，内容很简单，只有这些内容：

~~~c++
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
~~~

我们只需要将其编译为`.o`文件，然后在编译主文件的时候，只需要将其连接上去即可，至于其他需要使用这个文件的库，只需要包含`stb_image.h`头文件即可。我试过在其他文件中把上面的预编译器带上，以取代这个额外的文件，但是并不行，会报错。

至于Shader和Texture这两个库，是我自行写的，或者说抄的，作用分别是导入着色器文件和处理纹理，主要是为了简化代码。这两个库的设计还是有不少需要主要的点的，但是我并不准备附上代码做解释，看learnopengl上面的源码仿照着写就没问题。

编译的时候，我的策略是使用makefile，写的很垃圾，但是至少可以工作：

~~~makefile
header_folder = headers
lib_folder = lib
source_file = basic_texture.cpp
target = basic_texture
vpath %.h $(header_folder)
vpath %.o $(lib_folder)
vpath %.cpp $(lib_folder)


std_image = $(lib_folder)/stb_image.o
Shader = $(lib_folder)/Shader.o
Texture = $(lib_folder)/Texture.o
glad = $(lib_folder)/glad.o

$(target) : glad.h glfw3.h glad.o Shader.h Shader.o Texture.o stb_image.o
	g++ -I$(header_folder) -L$(lib_folder) -o $(target) $(source_file) $(std_image) $(glad) $(Shader) $(Texture) -lglfw3dll -lopengl32 -lglu32 -lgdi32

stb_image.o : stb_image.cpp stb_image.h
	g++ -I$(header_folder) -o $(std_image) -c $(lib_folder)/stb_image.cpp

Shader.o : Shader.cpp Shader.h
	g++ -I$(header_folder) -o $(Shader) -c $(lib_folder)/Shader.cpp

Texture.o : Texture.h Texture.cpp
	g++ -I$(header_folder) -o $(Texture) -c $(lib_folder)/Texture.cpp

.PHONY : clean

clean :
	rm $(target).exe
~~~

so，到目前为止，基本的着色器加载和纹理使用已经搞定了。

下一步是摄像机和trackball



### Assimp编译、使用与模型加载

前面说的Assimp的编译的确是不行的

首先，我们必须安装DirectX，去[这里](https://www.microsoft.com/en-us/download/details.aspx?id=6812)，下载安装，如果遇到`S1023`的安装错误，那么我们需要首先卸载`C++ Redistributable package(s) `2010版，然后就可以安装了

我现在的编译，直接使用Assimp的github最新版，而没有使用发行版，编译的过程中，还是首先使用cmake，然后再make，但是因为某些格式在make的过程中会出现错误，所以，我的策略是直接忽略这些格式方法是修改源码code文件夹下的`CMakeLists.txt`，这个文件控制着如何编译不同格式的Impoter，我添加了如下部分：

~~~cmake
set(ASSIMP_BUILD_IFC_IMPORTER FALSE)
set(ASSIMP_BUILD_GLTF_IMPORTER FALSE)
set(ASSIMP_BUILD_AMF_IMPORTER FALSE)
set(ASSIMP_BUILD_ASSBIN_IMPORTER FALSE)
set(ASSIMP_BUILD_COLLADA_IMPORTER FALSE)
set(ASSIMP_BUILD_IRRMESH_IMPORTER FALSE)
set(ASSIMP_BUILD_IRR_IMPORTER FALSE)
set(ASSIMP_BUILD_OGRE_IMPORTER FALSE)
set(ASSIMP_BUILD_XGL_IMPORTER FALSE)
set(ASSIMP_BUILD_X3D_IMPORTER FALSE)
set(ASSIMP_BUILD_3MF_IMPORTER FALSE)
~~~

位置的话，我放在了包含判断`ASSIMP_BUILD_ALL_IMPORTERS_BY_DEFAULT`的`MACRO`的后面

明显这样做是因为现在的版本太不友善了，我们只能选择是否按照默认编译所有的Impoter，如果不勾选将不编译任何的，这个做法很傻。

然后在cmake的config中我会取消勾选：

~~~
ASSIMP_BUILD_ASSIMP_TOOLS
ASSIMP_BUILD_ASSIMP_VIEW
ASSIMP_BUILD_TESTS
BUILD_SHARED_LIBS
~~~

然后要选上`ASSIMP_BUILD_ZLIB`

解释一下，取消的前三个明显是一些测试和检查的相关工具，没必要构建，并且构建的过程中似乎往往出问题

最后一个是构建动态链接库，明显我们想要的是静态链接

选上的ZLIB是一个叫做zlib的工具，跟压缩解压相关的，如果不构建，它将自动去系统里找，找到的往往是一些奇怪的，没法用，所以我们要自己构建。

然后，这样应该就没问题了

make的过程应该是基本不报错的，如果报错的话那就必须继续修改，直至make成功的构建了`libassimp.a`，一定是这个名字，然后中间如果报错的话，我们应该根据情况继续处理，如果是某个格式的Importer，就把它也忽略了

编译完成之后，我们应该在build的code里面可以找到`libassimp.a`这个静态库，另外我们还需要`contrib\zlib`里面的`libzlibstatic.a和zconf.h`文件，前者是我们需要的assimp库，明显。后两者就是我们构建zlib静态库和头文件，按道理说他们应该会被自动包含在assimp的静态库里面的，但是并没有，似乎我们也可以把它包含进去，但是我还没试

然后我们需要把头文件放在合适的地方，静态库放在合适的地方，针对特定的文件结构，我的一个编译命令是这样的：

`g++ model_loading.cpp -I. -L. glad.o stb_image.o -lglfw3dll -lopengl32 -lglu32 -lgdi32 -lassimp -lzlibstatic`

应该可以看出来我想说明什么吧。

这里还要说的是，之所以我用了zlib是因为在最后的使用编译中爆出了什么`_flatten_int_32`差不多吧，类似这个名字的错误，去网上搜就可以知道这是因为少了zlib

然后因为原本还报了跟irrXML相关的错，所以我把相关格式的importer也删了，所以可能有一些误伤，也许你可以尝试一下把`contrib\irrXML`里面的什么静态链接库也弄进去，也许就没必要删这么多格式了，至于可能还需要什么头文件之类的，大概就是这种思路，自行解决吧，感兴趣的话。



你以为到这里就结束了吗？naive

assimp的使用是一件巨复杂的事情，我们需要一些辅助的工具再从assimp的解析结果中构建出我们想要的数据结构，我主要参考learnopengl的教程和叫[王定桥的博主的教程](https://blog.csdn.net/wangdingqiaoit/article/details/52014321#comments)构建自己的model和mesh，我觉得后者写得还好一点，因为有了各种`has`相关的存在性判断什么的，但是遗憾的是我到现在都没成功，主要是模型不显示，原因未知，但是我使用learnopegl的github代码，做了一些文件结构调整，同时使用我自行编译的各种库让模型显示成功了，这说明库的编译是可以的，还是代码的原因，假以时日，做一些比对，一定可以发现问题所在。



我花费了大量的时间在查找错误所在，甚至重写了两遍我自己的mesh和model，但是无济于事，我甚至尝试了互相替换所有的文件，中间的过程简直让人有几分焦头烂额，但是，最终我找到了原因所在，令人难以置信，原因竟然是glm，我下载的glm似乎还是最新版本的，但是竟然不行，反而是learnopengl提供的20134版正常工作了，难以置信，我现在依旧在尝试查找glm到底是出了什么问题，同时将会逐步将所有的文件替换为我自己写的。



### 缓冲区的使用

这里主要指的是VAO,VBO,EBO

这三者分别是顶点数组，顶点缓存，索引缓存

他们的用处分别是管理一个对象，存储顶点数据，存储索引数据。VAO的存在主要用于多个对象的管理，如果存在多个对象，我们只需要为各自设置一个VAO，然后绑定某个对象的VAO，就可以绘制这个对象。VBO用于存储一个对象的所有顶点，EBO的作用是索引这些顶点，这样可以避免重复存储相同的顶点。

需要注意的问题就是，怎样使用这些东西，或者说次序。

每一个缓存都需要先生成一个缓存，然后绑定，然后存入数据，然后需要设置pointer，大致的过程是下面这样的：

~~~c++
unsigned int VAO, VBO, EBO;
glGenVertexArrays(1, &VAO);
glGenBuffers(1, &VBO);
glGenBuffers(1, &EBO);

glBindVertexArray(VAO);

glBindBuffer(GL_ARRAY_BUFFER, VBO);
glBufferData(GL_ARRAY_BUFFER, sizeof(Vertex)*mesh_vertexes.size(), &mesh_vertexes[0], GL_STATIC_DRAW);


glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, sizeof(unsigned int)*mesh_indices.size(), &mesh_indices[0], GL_STATIC_DRAW);

// vertex_position 0     normal 1   texture_position 2     tangent 3    bitangent 4
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)0);
// vertex normals


glBindVertexArray(0);
~~~

但是VAO要在先绑定之后，然后处理VBO和EBO，这样才能让VAO起作用。至于Pointer，无论是先enable还是先pointer都是可以的，只要是在绘制之前就行。

~~~c++
glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, 3 * sizeof(float), (void*)0);
/*第一个参数指定我们要配置的顶点属性。还记得我们在顶点着色器中使用layout(location = 0)定义了position顶点属性的位置值(Location)吗？它可以把顶点属性的位置值设置为0。因为我们希望把数据传递到这一个顶点属性中，所以这里我们传入0。
第二个参数指定顶点属性的大小。顶点属性是一个vec3，它由3个值组成，所以大小是3。
第三个参数指定数据的类型，这里是GL_FLOAT(GLSL中vec*都是由浮点数值组成的)。
下个参数定义我们是否希望数据被标准化(Normalize)。如果我们设置为GL_TRUE，所有数据都会被映射到0（对于有符号型signed数据是-1）到1之间。我们把它设置为GL_FALSE。
第五个参数叫做步长(Stride)，它告诉我们在连续的顶点属性组之间的间隔。由于下个组位置数据在3个float之后，我们把步长设置为3 * sizeof(float)。要注意的是由于我们知道这个数组是紧密排列的（在两个顶点属性之间没有空隙）我们也可以设置为0来让OpenGL决定具体步长是多少（只有当数值是紧密排列时才可用）。一旦我们有更多的顶点属性，我们就必须更小心地定义每个顶点属性之间的间隔，我们在后面会看到更多的例子（译注: 这个参数的意思简单说就是从这个属性第二次出现的地方到整个数组0位置之间有多少字节）。
最后一个参数的类型是void*，所以需要我们进行这个奇怪的强制类型转换。它表示位置数据在缓冲中起始位置的偏移量(Offset)。由于位置数据在数组的开头，所以这里是0。我们会在后面详细解释这个参数。*/
~~~





### 输入

这里记录一下glfw是怎么处理输入的

glfw的输入，我们一般而言关注的也就是键盘和鼠标，但是它的接口因为这些输入的输出数据的原因，被搞得有些复杂，我们基本上需要为键盘，鼠标位置，鼠标按钮，鼠标滚轮各自设置一个handler。

下面分别列出这些handler的写法：

1. 键盘：

   ~~~c++
   glfwSetKeyCallback(window, key_callback);
   
   void key_callback(GLFWwindow* window, int key, int scancode, int action, int mods)
   {
       if (key == GLFW_KEY_E && action == GLFW_PRESS)
           activate_airship();
   }
   ~~~

   这里的参数意义都比较明显，我们经常使用的也就是window, key, action而已，至于key的值是一系列的宏，详细列表[在这里](http://www.glfw.org/docs/latest/group__keys.html)

2. 鼠标位置：

   ~~~c++
   glfwSetCursorPosCallback(window, cursor_pos_callback);
   static void cursor_position_callback(GLFWwindow* window, double xpos, double ypos)
   {
   }
   ~~~

3. 鼠标按钮：

   ~~~c++
   glfwSetMouseButtonCallback(window, mouse_button_callback);
   void mouse_button_callback(GLFWwindow* window, int button, int action, int mods)
   {
       if (button == GLFW_MOUSE_BUTTON_RIGHT && action == GLFW_PRESS)
           popup_menu();
   }
   ~~~

   鼠标的按键也是一系列的宏，列表[在这里](http://www.glfw.org/docs/latest/group__buttons.html)

4. 鼠标滚轮：

   ~~~c++
   glfwSetScrollCallback(window, scroll_callback);
   void scroll_callback(GLFWwindow* window, double xoffset, double yoffset)
   {
   }
   ~~~

   我不清楚它为什么要设置一个xoffset，事实上有用的只有yoffset而已，并且yoffset的值只有-1或1而已。

我希望可以搞一个头文件什么的，把这些处理函数统一放进去，这一就可以免得每个源文件里面都要写一遍，但是问题在于某些情况下可能又会需要对每个事件做特殊的处理，所以，我不知道该怎么处理了。也许只能选择在每个源文件里面都写一遍了，但是对于特定的部分，可以编写一个另外的接口，这样就可以使得源文件里面的输入handler尽可能的简单了。例如，对于相机来说，可以额外写一个控制器。





### 下一步

最能吸引注意力的几个部分应该是：摄像机系统，模型载入，天空盒，光照

一直以来我使用的教程就是learnopengl，这个教程的质量可以说是相当高的，对于入门者而言。但是它的缺憾在于不是足够细致，例如：一开始就是窗口，三角形，着色器，纹理。这是让入门者可以快速写出点什么来，对opengl有一点感觉的方法，但是我的意思是，一旦入了一点门之后，可能我们还是会想要一点更加细致，系统和全面的知识。

这个时候，我推荐OpenGL超级宝典，俗称蓝宝书。原本，我尝试过使用这个来入门，但是完全云里雾里，看不懂。现在我发现，我可以了。

so，下一步我会花一些时间在这个上面，在基础上多花一点时间。



### 基础图元



#### 图元

七种图元

~~~
GL_POINTS           单点
GL_LINES            两个点组成一个线段
GL_LINE_STRIP	    依次组成连续线条
GL_LINE_LOOP        线圈
GL_TRIANGLES
GL_TRIANGLES_STRIP
GL_TRIANGLE_FAN
~~~

##### 点

~~~
void glPointSize(GLfloat size);
float sizes[2], step;

glGetFloatv(GL_POINT_SIZE_RANGE, sizes);
glGetFloatv(GL_POINT_SIZE_GRANULARITY, &step);
~~~

点总是正方形像素，要获得圆点只能使用抗锯齿模式进行绘制



##### 线



## 重启

又要重启了.....

要准备的事情好多，现在，即便只是从最基础的地方开始，我依旧想要一个着色器加载器，一个相机。

如果要写着色器加载器，我觉得我应该首先认真搞一搞着色器。然后是相机，相机的话这一次我要换四元数。也不知道搞不搞得定.........

### 着色器

最重要的两种着色器是顶点着色器和片段着色器，这已经是常识了嘛。

#### 数据类型

基本数据类型包括了：整数(signed/unsigned)，浮点数，布尔，返回类型也可以是void

组合类型包括了：向量类型，矩阵类型

##### 向量类型

向量类型根据基本的数据类型可以分为浮点向量，整数向量，无符号整数向量和布尔向量，根据分量数目可以分为2，3，4维向量，所以如下：

~~~
vec2,vec3,vec4: 浮点向量
ivec2,ivec3,ivec4: 整数向量
uvec2,uvec3,uvec4: 无符号整数向量
bvec2,bvec3,bvec4: 布尔向量
~~~

声明：

~~~GLSL
vec4 vertex = vec4(1.0f, 1.0f, 1.0f, 1.0f);
~~~

向量支持向量的操作，包括向量的加减，以及与标量乘除

索引方式：

~~~
vertex.xyzw
vertex.rgba
vertex.stpq

vertex.x = 1.0f;
vertex.xyz = vec3(1.0f, 1.0f, 1.0f);

//调换
vertex.zyxw = vertex1.xyzw;
~~~

意思就是每个分量都有名字了，很简单的吧。xyzw是顶点坐标名字，rgba是颜色通道名，stpq是纹理坐标名。任意向量可以用其中任何一种方式索引。

##### 矩阵类型

矩阵类型的特别之处在于只支持浮点数。根据维度不同，分为不同类型：

~~~
mat2, mat2x2: 2*2
mat3, mat3x3
mat4, mat4x4
mat2x3
mat2x4
....
~~~

矩阵可以认为是列向量组成的。

~~~
modelView[3] = vec4(1.0f, 1.0f, 1.0f, 1.0f);
//替换了第四列
vec3 s = model[3].xyz;
~~~

矩阵构造：

~~~
mat4 model = mat4(1.0f, 1.0f, 1.0f, 1.0f,
                  1.0f, 1.0f, 1.0f, 1.0f,
                  1.0f, 1.0f, 1.0f, 1.0f,
                  1.0f, 1.0f, 1.0f, 1.0f);
                  
mat4 model = mat4(1.0f); //单位矩阵
~~~

#### 存储限定符

变量都可以限定存储类型。主要分为：

~~~
<none>      默认本地变量
const       常量
in          从前一个阶段获取的
out         到下一个阶段的
in centroid     上个阶段，质心插值
out centroid    下个阶段，质心插值
inout       in/out，局部函数参数
uniform     从客户端代码传过来的，顶点间不做改变
~~~

我之前最常接触的也就是in/out/uniform

因为GLSL不支持指针，所以如果允许函数内部修改传入的参数，并且修改可以反馈到外面，就应该用inout，这种类型也只用于这里

至于`centroid noperspective smooth`等基本用于插值，现在我应该用不上



#### Uniform值

uniform就是不变值，设置方法需要先获取，然后设置，就是`glGetUniformfromLocation`这样的函数，设置函数等等就不在这里说了。



至于内置函数，非常非常多了，就不说了。

### 着色器加载器

加载器其实很简单，只是从路径读入着色器文件，设置着色器，同时提供一些设置参数的代码。

我决定仿效`LearnOpenGL`的代码，然后同时提供尽可能全面的功能，主要还是抄它的代码了，只是修改一下我想改的地方。

也写成头文件形式就好。



### 摄像机

现阶段，我很想直接抄一个完美的相机，然后完事。可是我却发现，之前的关于坐标变换的只是我已经全部忘了，而找到的那个相机的代码又十分复杂，我愈发觉得如果不好好复习一下相机和坐标变换我根本搞不定。

所以，又要从头来一遍坐标和相机了。

#### 空间与坐标变换

5种重要的坐标系应该还没忘，局部空间，世界空间，相机空间，剪裁空间，屏幕空间。

经过四个坐标变换我们就能知道放在一个特定世界坐标上的点应该呈现在屏幕的哪个位置。

这四个坐标变换对应四个变换矩阵，依次是`model, view, projection, viewport transform`，最后一个可以由opengl完成，所以，我们要关心的就是3个矩阵。

其中model是决定把模型怎么放在世界中，这个可以非常简单。所以也不用非常关心

然后投影矩阵负责完成剪裁，至于这里面的正投影和透视投影也很简单就不提了，反正我一般都用透视投影嘛。至于透视投影，其实就是为了将一个平截头体3维空间内的点映射到一个剪裁空间内。这一步只需要定义出来近点，远点，fov，宽高比，实际上直接就可以推算出公式，换句话说有了这几个参量就可以直接计算出projection matrix。也就是说这个矩阵也不需要考虑。

总而言之，只需要考虑如何构建一个View Matrix就行了，它也就决定了摄像机。

而这个坐标变换的目的自然就是将世界空间转换到相机空间中，所谓相机空间就是以相机的指向的逆向作为Z轴正向，上作为Y轴正向，右作为X轴正向的空间，所以根据线代的只是就是找到相机的正交基在世界坐标中的表示，具体的我有点忘记了。

但是要点其实就是找到相机的正交基。



其实分解一下，相机的基也不是很难找的样子，它包含了平移和旋转两种变换，任意一个相机坐标和指向只需要首先完成相机在原点的旋转，然后平移到指定位置即可。

这其中平移操作尤为简单，可以不做考虑。所以核心问题就转换为了如何完成旋转操作。

之前使用的方法一直都是欧拉角：俯仰角，偏转角，翻滚角(pitch yaw roll)，会出现万向节锁，还要考虑操作次序等等。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      

四元数是包含了3个虚数分量和一个实数分量的虚数，`[x, y, z, w]`，一个虚数可以表示一个旋转轴和一个旋转角度。

据我目前的调研来说，使用四元数的相机其实还是那样，同样使用了各种平移操作，加上俯仰，翻滚和偏航，只是在翻滚和偏航和俯仰的时候是使用四元数进行计算而已。

然后我又对比了一下[这个视频](https://www.youtube.com/watch?v=NPNX6HxQqRA)，这一次我意识到我为什么想要这个视频里面的效果了，以及它有什么不同之处。

其实观察可以发现他所做的操作与普通定义下的相机没有区别，同样包含了平移操作，以及pitch,yaw,roll，不同之处在于yaw的处理，明显，按照正确的方式应该如何定义yaw？自然是相机绕自身的y轴旋转，这种情况下相当于左右环视，就是一般的效果，而这个视频里的效果明显不同，我认为他应该是在做了平移操作之后并没有立即更新y轴，于是产生了绕物体环视的效果，而这个恰好就是我想要的。

花了好多时间，终于把上面视频里的相机接入我的代码了，可是并未能实现绕一个圆轨道运行的效果。什么鬼。接下来，我也做了巨多的尝试，试图搞懂问题所在，我还掌握了一种技术，可以在屏幕上实时绘制一条表示某个向量的线，这样就能把旋转轴实时显示出来，在这个的帮助下，我还是没能找出问题所在，但是我越来月意识到似乎是我的思路一直很不清晰，我似乎并没有真的懂坐标变换和相机，我需要好好考虑考虑。



其实在上面的实时绘制向量的过程我就发现了，这些基础知识我掌握的情况还很差，所以，要不相机就先凑合着用吧，有空就巩固基础知识先。至于相机用哪个嘛？要不就先还是`LOG`的先(以后就用LOG代指learnopengl了)

