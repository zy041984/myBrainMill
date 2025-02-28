# [VC调用giflib（1）:VC编译giflib](https://www.cnblogs.com/stronghorse/p/16002872.html "发布于 2022-03-14 09:39")

作者：马健  
邮箱：[stronghorse_mj@hotmail.com  
](mailto:stronghorse_mj@hotmail.com)主页：[http://www.comicer.com/stronghorse](http://www.comicer.com/stronghorse)  
发布：2020.03.14

以前我只与静态GIF文件打交道，用CxImage就够了。最近需要编、解码动画GIF，才发现CxImage不够用。刚好处理webp动画的libwebp库提供了用giflib解码动画GIF的源代码，所以就试着在我的VC代码中调用giflib，结果发现实在麻烦——这个库的官方文档基本没有，官方例子代码也绕来绕去，网上的说明和例子不仅不多，还到处是坑，说起来都是泪。好在经过一番折腾，我想要的功能最终还是实现了，虽然过程曲折了一点。为了不在同样的坑里踩两遍，于是写下了这一系列笔记。

**本系列笔记所谈的giflib均针对5.2.1版本，不再逐一说明。**

==========================================================================

要在VC里调用giflib，第一步当然是用VC编译giflib。考虑到复用性，以编译成静态库为宜。网上有文章介绍这个编译过程，但其中可能藏坑。这里先列出我已经填过坑的编译过程：

1、新建一个空项目，名称为giflib，类型为静态库（Static library），注意务必把预编译头（Precompiled header）选项去掉。  
  
2、把创建出来的项目文件剪切到giflib-5.2.1文件夹。  
  
3、将h和c文件添加到项目中，包括：  
dgif_lib.c  
egif_lib.c  
gif_err.c  
gif_hash.c  
gifalloc.c  
openbsd-reallocarray.c  
qprintf.c  
gif_hash.h  
gif_lib.h  
gif_lib_private.h

其中：  
1）加入qprintf.c是因为我要调用giftext.c中的代码。如果不调用，可以不加入。  
2）如果要使用giflib的调色板量化功能（用法参见例子gif2rgb.c），还需要加入quantize.c。我嫌它这个中位切分（median cut）算法太low就没加，改用FreeImage提供的其他调色板量化算法。  
3）、如果要编译giflib的命令行，还需要从github开源项目  
[https://github.com/bjornblissing/osg-3rdparty-cmake.git](https://github.com/bjornblissing/osg-3rdparty-cmake.git)  
的giflib文件夹下载这两个命令行解析代码，加入工程中：  
getopt.c  
getopt.h  
我不打算编译生成命令行工具，所以就没加。  
  
4、设置Code Generation、Output，创建x64编译配置。

5、开始编译。编译报错时先把#include <stdint.h>、#include<unistd.h>代码全部注释掉。VC没有这两个头文件，也不需要。  
  
6、将gif_lib.h中  
`#include <stdbool.h>`  
这一行，改成
```
//#include 
#ifndef __cplusplus
typedef char bool;
#define false 0
#define true 1
#endif
typedef unsigned __int32 uint32_t;
```
即注释掉对stdbool.h的引用，在C下定义bool量为单字节类型，同时增加uint32_t类型定义。注意网上的一些giflib编译说明，包括上面那个github开源项目提供的stdbool.h中一般是  
typedef int bool;  
即把bool说明为4字节的int类型，而不是我上面说的单字节char类型。这就是我要填的第一个坑，本文后面再详谈。  
  
7、对giflib源代码进行清理，以消除一些警告错误：  
1）egif_lib.c中的EGifPutPixel、EGifCompressLine、EGifCompressOutput函数原型的参数类型与实现不一致，按照实现改原型。  
2）EGifOpenFileName、EGifOpenFileHandle、DGifOpenFileName、DGifOpenFileHandle等函数会报一大堆4996警告，在gif_lib.h的  
`#ifdef __cplusplus`  
前面加一行  
`#pragma warning( disable : 4996 )`  
世界就清净了。  
3）编译x64版本还有几个类型转换警告，强制转换即可，不转也无所谓。  
4）如果需要处理Unicode文件名，把EGifOpenFileName、DGifOpenFileName函数复制一份并更名为EGifOpenFileNameW、DGifOpenFileNameW，把函数原型中FileName参数的类型从char*改成wchar_t*，函数体中的open改成_wopen即可。当然也可以不这么麻烦，而是用EGifOpen、DGifOpen解决Unicode文件名的问题，下一篇笔记会专门讲这两个函数的使用。

经过以上修改，不出意外就可以成功编译出giflib静态库。至于giflib源代码中的内存泄漏（memory leak）等坑，等后面讲到的时候再详谈。

这里先说详细谈一下前面提到的第一个坑，这个坑可以细化为以下三个问题：

1. 为什么giflib要单独定义bool类型？
2. bool类型究竟是1字节还是4字节？
3. 字节数不同对于giflib的调用究竟有什么影响？

先回答第1个问题：bool是C++中规定的内建（built-in）数据类型，在C中则不是。所以纯C代码或跨平台库，通常要么就不使用bool类型，要么就自己定义一个，最著名的就是Windows SDK里单独定义的BOOL类型（typedef int BOOL），freetype-2.8里则定义了FT_Bool数据类型（typedef unsigned char FT_Bool），jpeg_v6b中定义了boolean（typedef unsigned char boolean），djvulibre-3.5.28的定义和我的差不多：
```
/* - BOOL */
#if !defined(HAVE_BOOL) && !defined(bool)
#define bool  char
#define true  1
#define false 0
#endif    
```
giflib是纯C代码，所以就用了C99标准库stdbool.h，对bool类型进行定义。VC对C99标准只是部分实现，就没有这个库，所以需要自己搞。

再回答第2个问题：在VC 2010帮助文档的bool keyword[C++]词条里，前面引用的都是C99标准说明文字，后面来了一段神转折：

**Microsoft Specific**

In Visual C++4.2, the Standard C++ header files contained a typedef that equated bool with int. In Visual C++ 5.0 and later, bool is implemented as a built-in type with a size of 1 byte. That means that for Visual C++ 4.2, a call of sizeof(bool) yields 4, while in Visual C++ 5.0 and later, the same call yields 1. This can cause memory corruption problems if you have defined structure members of type bool in Visual C++ 4.2 and are mixing object files (OBJ) and/or DLLs built with the 4.2 and 5.0 or later compilers.

也就是说，从VC 5.0开始，微软就认定用一个字节表示bool类型足够了。在MSDN在线文档  
[Built-in types (C++) | Microsoft Docs](https://docs.microsoft.com/en-us/cpp/cpp/fundamental-types-cpp?view=msvc-170#sizes-of-built-in-types)  
中，也说明bool就是一个字节，而不是像网上一些代码所定义的是int类型，即4个字节。

结合以上两个问题的答案就可以回答第3个问题：

1. giflib是纯C代码，用它编译出来的静态库如果只被纯C代码调用，大家使用相同的stdbool.h定义，对bool类型的理解一致，那么P事没有，一切正常。
2. 如果是从VC写的C++代码（CPP）中调用giflib，就会出现数据类型不匹配。

具体而言，由于VC在C++中认定bool只有一个字节，则在调用giflib函数时，对于bool类型就只会传递1个字节的值，但giflib如果认定bool有4个字节，那么接收bool类型参数时就按4字节接收，最终所收到的bool类型参数只有最低1个字节是有效的，其余高位3字节具体是这么值，完全看临近的变量是什么值。

要验证这个问题，其实很简单：

1. 就按网上一般说的，采用上面那个github开源项目提供的stdbool.h，把bool定义为int类型，编译giflib静态库。
2. 用VC写一段CPP代码，在调用EGifOpenFileName创建一个GIF文件后，就调用函数  
    `void EGifSetGifVersion(GifFileType *GifFile, const bool gif89);` 
    对于gif89参数传递一个false，然后跟踪进去，实际看一下giflib收到的是不是0。

除了这个接口外，最最关键的是GifImageDesc结构体的Interlace成员也有这个问题。我也是在发现无论我怎么设置，创建出来的都是交错GIF文件时，才发现双方在bool类型上是鸡同鸭讲。
# [VC调用giflib（2）:EGifOpen、DGifOpen用法](https://www.cnblogs.com/stronghorse/p/16002879.html "发布于 2022-03-14 09:41")

作者：马健  
邮箱：[stronghorse_mj@hotmail.com  
](mailto:stronghorse_mj@hotmail.com)主页：[http://www.comicer.com/stronghorse](http://www.comicer.com/stronghorse)  
发布：2020.03.14

早期版本的giflib对GIF编码、解码均只提供针对文件的接口，如解码的DGifOpenFileName、DGifOpenFileHandle函数。从v5开始增加了EGifOpen、DGifOpen函数，可以更灵活地操作GIF文件，如解码内存中的GIF文件、在内存中编码GIF文件，或打开Unicode文件名的文件。

giflib官方文档对这两个新增接口说得含含糊糊，网上也有人在问究竟应该怎么用，我也是绕了一下才搞清楚，所以在这里记录一下。

这两个接口的核心思想是，把读、写操作变成函数指针，具体是在内存还是在文件中读、写，完全看函数指针所指的函数是怎么实现的。dgif_lib.c中的InternalRead函数、egif_lib.c中的InternalWrite函数就提供了读、写函数的例子，看懂了其中针对FILE*的操作，即可在此基础上写出自己的读、写函数。

例如，如果想解码内存缓冲区中的GIF文件，则可以这样写代码：
```
struct GifData {
	const unsigned char* m_lpBuffer;
	size_t m_nBufferSize;
	size_t m_nPosition;
};

// copied from CMemFile::Read
int InternalRead_Mem(GifFileType *gif, GifByteType *buf, int len) {
	if (len == 0)
		return 0;

	GifData* pData = (GifData*)gif->UserData;
	if (pData->m_nPosition > pData->m_nBufferSize)
		return 0;

	UINT nRead;
	if (pData->m_nPosition + len > pData->m_nBufferSize || pData->m_nPosition + len < pData->m_nPosition)
		nRead = (UINT)(pData->m_nBufferSize - pData->m_nPosition);
	else
		nRead = len;

	memcpy((BYTE*)buf, (BYTE*)pData->m_lpBuffer + pData->m_nPosition, nRead);
	pData->m_nPosition += nRead;

	return nRead;
}

// 解码内存缓冲区中的GIF文件，文件首地址pBuffer，文件长度dwLen，结果存入pImage
BOOL LoadGifFromBuffer(BYTE* pBuffer, CxImage* pImage, DWORD dwLen)
{
	// DGifOpen要的数据结构
	GifData data;
	memset(&data, 0, sizeof(data));
	data.m_lpBuffer = pBuffer;
	data.m_nBufferSize = dwLen;

	// 打开GIF文件
	GifFileType* gif = DGifOpen(&data, InternalRead_Mem, NULL);
	……
}
```

如果基于MFC的CFile类，则可以一次性解决Unicode文件名和在内存中对GIF文件进行操作的问题，如下面的代码：
```
// 写GIF文件内容到CFile
int InternalWrite_CFile(GifFileType *gif, const GifByteType *buf, int len) {
	CFile* pData = (CFile*)gif->UserData;
	pData->Write(buf, len);
	return len;
}

// 生成GIF文件，pszTgtPath是目标路径
BOOL MakeGif(LPCTSTR pszTgtPath, ……)
{
	// 打开输出的 GIF 文件  
　　　　 CFile fileOut;  
　　　　 GifFileType* GifFile;
	if (!fileOut.Open(pszTgtPath, CFile::modeCreate|CFile::modeWrite))
		return FALSE;
	if ((GifFile = EGifOpen(&fileOut, InternalWrite_CFile, NULL)) == NULL)
		return FALSE;
	……
}
```
如果在上面的MakeGif函数里，把CFile换成CMemFile，使用同样的InternalWrite_CFile函数，则可以实现在内存里生成GIF文件。
# [VC调用giflib（3）:GIF文件编、解码](https://www.cnblogs.com/stronghorse/p/16002884.html "发布于 2022-03-14 09:43")

作者：马健  
邮箱：[stronghorse_mj@hotmail.com  
](mailto:stronghorse_mj@hotmail.com)主页：[http://www.comicer.com/stronghorse](http://www.comicer.com/stronghorse)  
发布：2020.03.14

**一、GIF解码**

用giflib对GIF文件进行解码有两个流派：

1. 自己循环调用DGifGetRecordType，读到一帧就解码、显示一帧，具体例子可以参见gif2rgb.c中的GIF2RGB函数。该流派实现的时候辛苦一点，但可以边解码、边显示，所以用户反应更快，做得好的话也更省内存。
2. 直接调用DGifSlurp，一次解码所有帧，然后再慢慢对各帧进行处理、显示。具体例子可以参见libwebp下的例子代码anim_util.c，其中的ReadAnimatedGIF函数。

libwebp下的这个ReadAnimatedGIF函数，可能是我见过最用心的解动画GIF代码，而且与解动画webp兼容，所以我一见就喜欢。如果非要说有什么瑕疵的话，可能就是google程序员还是太年轻了一点，对社会之复杂、道德之沦丧、人性之卑劣认识不足，还需要在
```
if (image->canvas_width == 0 || image->canvas_height == 0) {  
    ……  
  }
```
 这个块后面增加一段
```
// 对画布尺寸进行修正：某些GIF的SWidth、SHeight小于帧的宽、高
	for (int i = 0; i < frame_count; i++)
	{
		const GifImageDesc& ImageDesc = gif->SavedImages[i].ImageDesc;
		if ((ImageDesc.Width + ImageDesc.Left) > gif->SWidth)
			gif->SWidth = ImageDesc.Width + ImageDesc.Left;
		if ((ImageDesc.Height + ImageDesc.Top) > gif->SHeight)
			gif->SHeight = ImageDesc.Height + ImageDesc.Top;
	}	
```
我真不是在讲笑话——我从网上下载的一些GIF文件是真有这个问题，而且数量不少。碰到这种GIF文件头与帧头数据不一致的情况，CxImage是直接退出解码，老牌的商业版看图软件ACDSee 2022旗舰版则是按照画布尺寸显示一个小黑方块，libwebp如果不做上述修正就直接缓冲区溢出了。

**二、GIF编码**

与解码一样，用giflib对GIF进行编码也有两个流派：

1. 按照帧数打循环，对于每一帧，先EGifPutImageDesc，然后循环各像素行EGifPutLine。很辛苦，但编码后的数据直接进文件，不会在内存里磨磨唧唧。参见gif2rgb.c中的SaveGif函数。
2. 先在内存里组帧，然后调用EGifSpew函数一次写到文件里。内存占用略多，但更灵活。网上搜索EGifSpew能找到一些使用的例子，此处从略。

需要注意的是，如果网上找到的代码在插入帧时是这个路数：
```
GifImageDesc *imageDesc = (GifImageDesc *) malloc(sizeof(GifImageDesc));
……
image->ImageDesc = *imageDesc;

GraphicsControlBlock *GCB = (GraphicsControlBlock *) malloc(sizeof(GraphicsControlBlock));
……
EGifGCBToSavedExtension(GCB, GifFile, i);
```
那么就一定要自己free掉所分配的imageDesc、GCB指针，否则就会产生内存泄漏。其实上面的动态内存分配根本就没道理：需要分配的内存长度是定长，而且没几个字节，giflib还自动对缓冲区内容进行备份，不需要长期保存，所以直接使用局部变量更省事，就像这样：
```
GifImageDesc imageDesc;
……
image->ImageDesc = imageDesc;

GraphicsControlBlock GCB;
……
EGifGCBToSavedExtension(&GCB, GifFile, i);	
```
另外giflib原版的EGifSpew函数中存在严重的内存泄漏，但giflib的内存泄漏不止这一处，所以在下一篇笔记中专门说这个事情。
但在我看来，giflib对动画GIF文件的编码只是解决了LZW数据压缩的问题，即只能实现最基本的GIF文件编码、写入功能，但以下关键功能是缺失的：

**1、颜色量化、抖动**

giflib生成的GIF文件最多只能有256色，因此在真彩图像转GIF时，就需要进行颜色量化，即把真彩图像所使用的宽广色域，合理地减少到256色，甚至更少的色。具体可参见百度百科“颜色量化算法”词条：[颜色量化算法_百度百科 (baidu.com)](https://baike.baidu.com/item/%E9%A2%9C%E8%89%B2%E9%87%8F%E5%8C%96%E7%AE%97%E6%B3%95)

不过这个词条中所列的算法都太老了。giflib的quantize.c就实现了其中的中位切分（median cut）算法，结果连giflib的作者自己都嫌弃，甚至一度被移出giflib的源代码，见文件头的注释。

我个人比较喜欢FreeImage库实现的Xiaolin Wu算法和神经网络算法。在256色时各种算法的差距可能不太明显，但在减色到更少的色彩数，比如16色或16色以下时，不同算法的结果可能差距巨大。

在把宽广色域量化减少到较少的颜色数时，最常造成的一个不良后果是“等高线”现象，即原先色彩丰富、过渡自然的地方，减色后就成了界限明显的色块，看起来就像地图上的等高线。而解决这个问题的手段就是抖动（dither）。

如果经常看网上视频转出来的GIF动图，就可以看出抖动对转换软件的影响——以前的转换软件没有抖动功能，所以转出来的动图有明显的等高线（色块），后来采用有序抖动(ordered bdithering)，等高线看不到了，但Bayer矩阵形成的网点却很明显，现在则基本上都采用误差扩散（error diffusion）抖动算法，已经基本上看不出网点，视觉效果大大改善。

对于大多数彩色图像而言，我认为误差扩散抖动采用最经典的Floyd–Steinberg矩阵就够了。需要注意的是，中文网页中有一些采用的是二阶假冒Floyd–Steinberg矩阵，擦亮眼睛别被骗了。另外减色、抖动的时候需要尽量避免图像透明区域的影响，尤其是抖动的时候，如果把透明区域的背景色也抖进来就好看了。

**2、计算各帧共享的全局调色板**

按照GIF标准规定，动画GIF中的各帧可以共享一个全局调色板，也可以每一帧都有自己的调色板。

当各帧颜色相差不大时，共享全局调色板可以减小最终GIF文件的长度。以256色为例，调色板长度是0.75 KB，在使用全局调色板的情况下，不论图像是10帧还是20帧，调色板占用空间就是0.75 KB。如果各帧都有自己的256色调色板，则10帧占用的调色板空间就是7.5 KB，20帧就是15 KB。

如果各帧之间颜色差距明显，则使用共享调色板就会影响图像效果，即相当于在颜色差异明显的情况下强行统一了颜色。另外如果有些帧的颜色数明显小于其他帧，这些帧也不宜采用全局调色板。例如某动画GIF大多数帧都可以共享256全局色调色板，但开头、结尾的帧可能因为过渡的需要只有背景色或少数几种色，，这时如果这些帧采用自己的调色板，则一方面因为颜色数较少，调色板占用不了多少空间，另外一方面可以有效压缩像素编码位数（2色只需1 bit，4色只需2 bit，而256色需8 bit），从而降低编码后的帧数据长度。

所以好的动画GIF生成软件，既要能计算出各帧共享的全局调色板，又要能具体分析每一帧究竟应该使用公共调色板，还是应该用自己的私有调色板。

我自己是在计算出全局调色板后，在往GIF里插帧的时候再看该帧是否适用全局调色板，不适用就另外计算该帧的调色板。如果全部帧都有自己的调色板，则在存盘前删除全局调色板。所以我只能先在内存里组帧，再调用EGifSpew函数一次性存盘。

**3、运动检测**

运动检测是指从视频、动画中识别出发生变化或移动的区域，这样在动画GIF中，后一帧只需要存储相对前一帧发生了变化的区域，没变化的区域就不用存储，从而可以有效压缩GIF文件长度。

很明显giflib中没有提供运动检测功能，有兴趣的可以去看libwebp源代码，他家生成动画webp的时候做了运动检测。

我自己懒得去扒libwebp的源代码，所以做了一个最简单的：根据每一帧图像的透明区域对帧图像进行裁剪，只存非透明区域的图像。压缩效果肯定不如运动检测，但比没有强。

**4、可变帧率**

GIF文件格式允许对每一帧单独指定帧间隔，这样如果动画GIF制作软件比较有理想、有追求，就可以在画面变化激烈时缩小帧间隔，在画面变化不大时延长帧间隔，相当于视频中的可变码率，也能有效压缩最终GIF文件长度。

giflib中仍然没有提供这个功能。我也懒得扒代码，而是继续滑向无耻的深渊：如果静态图像系列转动画GIF，简单点就用固定帧率；如果动画webp直接转GIF，就把webp的帧率直接复制过来用。
# [VC调用giflib（4）:内存泄漏与功能缺失](https://www.cnblogs.com/stronghorse/p/16002888.html "发布于 2022-03-14 09:43")

作者：马健  
邮箱：[stronghorse_mj@hotmail.com  
](mailto:stronghorse_mj@hotmail.com)主页：[http://www.comicer.com/stronghorse](http://www.comicer.com/stronghorse)  
发布：2020.03.14

**一、EGifSpew的内存泄漏与功能缺失**

在上一篇笔记里说过，我因为要计算全局调色板和局部调色板，所以只能用EGifSpew对GIF进行编码，但giflib中对这个函数的实现存在严重的内存泄漏问题。

**1）EGifPutScreenDesc函数调用造成的内存泄漏**

在EGifSpew函数开头部分是这样调用EGifPutScreenDesc函数的：
```
int
EGifSpew(GifFileType *GifFileOut) 
{
    int i, j; 

    if (EGifPutScreenDesc(GifFileOut,
                          GifFileOut->SWidth,
                          GifFileOut->SHeight,
                          GifFileOut->SColorResolution,
                          GifFileOut->SBackGroundColor,
                          GifFileOut->SColorMap) == GIF_ERROR) {
        return (GIF_ERROR);
    }
	……
```
而在EGifPutScreenDesc函数中，是这样操作的：
```
int
EGifPutScreenDesc(GifFileType *GifFile,
                  const int Width,
                  const int Height,
                  const int ColorRes,
                  const int BackGround,
                  const ColorMapObject *ColorMap)
{
    GifByteType Buf[3];
    GifFilePrivateType *Private = (GifFilePrivateType *) GifFile->Private;
    const char *write_version;
    GifFile->SColorMap = NULL;
    
    ……

    if (ColorMap) {
        GifFile->SColorMap = GifMakeMapObject(ColorMap->ColorCount,
                                           ColorMap->Colors);
        ……
    } else
        GifFile->SColorMap = NULL;
```
即一进`EGifPutScreenDesc`函数，三不管就把`GifFile->SColorMap`赋空了，后面再重新给它分配内存。而在我的源代码中，是先给`GifFile->SColorMap`分配了全局调色板的，结果进去后就成了野指针。这个操作够不够骚？说实话我刚看到的时候真的是被雷到了，不由感慨我还是太年轻，这世上还有无数的奇葩我还没见识过。

解决办法就是在调用`EGifPutScreenDesc`函数前先对`GifFile->SColorMap`指针进行备份，调用后释放掉即可。

**2）EGifCloseFile函数调用造成的内存泄漏**

一般使用EGifSpew函数生成动画GIF的过程，都是先把各帧图像存储到GifFileType结构体的SavedImages数组中，然后再调用EGifSpew把数组中的各帧顺序编码、存盘。但是EGifSpew的问题就在于各帧编码完成后，并没有释放SavedImages数组中的各帧内存，直接就调用EGifCloseFile函数释放掉GifFileType结构体，结果SavedImages数组中的各帧就成了没爹没娘的野指针。

解决的办法就是在调用EGifCloseFile之前，先调用GifFreeSavedImages释放掉SavedImages数组中分配的内存。

除了上述内存泄漏问题外，EGifSpew函数中还存在一个功能缺失：在EGifPutScreenDesc后，没有写入动画GIF的循环次数，导致生成的GIF文件只能动一次。正常情况下循环次数是写在帧数据前面的，所以应该在EGifPutScreenDesc之后，在for循环写各帧数据之前。

最后我实在是没办法，只能把原版EGifSpew函数复制一份出来修改成MyEGifSpew，完整代码如下：
```
// MyEGifSpew要用到此函数，但在egif_lib.c中此函数是static局部函数，所以照抄一遍
static int
EGifWriteExtensions(GifFileType *GifFileOut, 
					ExtensionBlock *ExtensionBlocks, 
					int ExtensionBlockCount) 
{
	if (ExtensionBlocks) {
		int j;

		for (j = 0; j < ExtensionBlockCount; j++) {
			ExtensionBlock *ep = &ExtensionBlocks[j];
			if (ep->Function != CONTINUE_EXT_FUNC_CODE)
				if (EGifPutExtensionLeader(GifFileOut, ep->Function) == GIF_ERROR)
					return (GIF_ERROR);
			if (EGifPutExtensionBlock(GifFileOut, ep->ByteCount, ep->Bytes) == GIF_ERROR)
				return (GIF_ERROR);
			if (j == ExtensionBlockCount - 1 || (ep+1)->Function != CONTINUE_EXT_FUNC_CODE)
				if (EGifPutExtensionTrailer(GifFileOut) == GIF_ERROR)
					return (GIF_ERROR);
		}
	}

	return (GIF_OK);
}

// 原版的EGifSpew存在内存漏洞，而且没有设置循环播放次数，导致生成的GIF只能播放一次
// nLoopTimes：循环播放次数，0表示无限循环播放
static int MyEGifSpew(GifFileType *GifFileOut, int nLoopTimes) 
{
	int i, j; 

	// EGifPutScreenDesc会破坏GifFileOut->SColorMap，所以调用前必须先备份，否则出现野指针
	ColorMapObject* SColorMap = GifFileOut->SColorMap;
	if (EGifPutScreenDesc(GifFileOut,
		GifFileOut->SWidth,
		GifFileOut->SHeight,
		GifFileOut->SColorResolution,
		GifFileOut->SBackGroundColor,
		GifFileOut->SColorMap) == GIF_ERROR) {
			return (GIF_ERROR);
	}
	// 在EGifPutScreenDesc中重新分配了GifFileOut->SColorMap，所以手工释放前面保存的
	GifFreeMapObject(SColorMap);

	// 在共享调色板之后，写入循环次数。原版EGifSpew少了这一项，所以生成的GIF只能动一次
	{
		/* Create a Netscape 2.0 loop block */
		unsigned char params[3] = {1, 0, 0};	// 后两个字节是循环次数，0表示无限循环
		params[1] = (nLoopTimes & 0xff);
		params[2] = (nLoopTimes >> 8) & 0xff;
		if (EGifPutExtensionLeader(GifFileOut, APPLICATION_EXT_FUNC_CODE) != GIF_OK ||
			EGifPutExtensionBlock(GifFileOut, 11, "NETSCAPE2.0") != GIF_OK ||
			EGifPutExtensionBlock(GifFileOut, 3, params) != GIF_OK ||
			EGifPutExtensionTrailer(GifFileOut) != GIF_OK
			)
			return (GIF_ERROR);
	}

	for (i = 0; i < GifFileOut->ImageCount; i++) {
		SavedImage *sp = &GifFileOut->SavedImages[i];
		int SavedHeight = sp->ImageDesc.Height;
		int SavedWidth = sp->ImageDesc.Width;

		/* this allows us to delete images by nuking their rasters */
		if (sp->RasterBits == NULL)
			continue;

		if (EGifWriteExtensions(GifFileOut, 
			sp->ExtensionBlocks,
			sp->ExtensionBlockCount) == GIF_ERROR)
			return (GIF_ERROR);

		if (EGifPutImageDesc(GifFileOut,
			sp->ImageDesc.Left,
			sp->ImageDesc.Top,
			SavedWidth,
			SavedHeight,
			sp->ImageDesc.Interlace,
			sp->ImageDesc.ColorMap) == GIF_ERROR)
			return (GIF_ERROR);

		if (sp->ImageDesc.Interlace) {
			/* 
			* The way an interlaced image should be written - 
			* offsets and jumps...
			*/
			int InterlacedOffset[] = { 0, 4, 2, 1 };
			int InterlacedJumps[] = { 8, 8, 4, 2 };
			int k;
			/* Need to perform 4 passes on the images: */
			for (k = 0; k < 4; k++)
				for (j = InterlacedOffset[k]; 
					j < SavedHeight;
					j += InterlacedJumps[k]) {
						if (EGifPutLine(GifFileOut, 
							sp->RasterBits + j * SavedWidth, 
							SavedWidth)	== GIF_ERROR)
							return (GIF_ERROR);
				}
		} else {
			for (j = 0; j < SavedHeight; j++) {
				if (EGifPutLine(GifFileOut,
					sp->RasterBits + j * SavedWidth,
					SavedWidth) == GIF_ERROR)
					return (GIF_ERROR);
			}
		}
	}

	if (EGifWriteExtensions(GifFileOut,
		GifFileOut->ExtensionBlocks,
		GifFileOut->ExtensionBlockCount) == GIF_ERROR)
		return (GIF_ERROR);

	// 原版EGifSpew少了这一句，导致大量的内存漏洞
	GifFreeSavedImages(GifFileOut);

	if (EGifCloseFile(GifFileOut, NULL) == GIF_ERROR)
		return (GIF_ERROR);

	return (GIF_OK);
}
```
上面的代码修正了两处内存泄漏，并允许调用时设置循环次数。在写入循环次数的时候，使用了最流行的NETSCAPE2.0，没有用比较少见的ANIMEXTS1.0。

另外如果是对细节比较在意的程序员，在调用EGifSpew开始写GIF文件之前，还应该调用EGifGetGifVersion函数，让giflib自动设置输出的GIF文件的版本。虽然不调用也不会有啥大影响，但细节终归是细节。

**二、EGifPutImageDesc函数中的内存泄露**

这个内存泄漏比较隐蔽。先看该函数中的一段代码：
```
if (ColorMap != GifFile->Image.ColorMap) {
		if (ColorMap) {
		    // 用ColorMap更新GifFile->Image.ColorMap
		    ……
		} else {
		    GifFile->Image.ColorMap = NULL;
		}
}
```
如果满足第一层条件`ColorMap != GifFile->Image.ColorMap`，说明这两个指针总有一个非空，第二层判断的else块中ColorMap为空情况下，`GifFile->Image.ColorMap`就必然不为空，这个时候把它直接赋空，妥妥的内存泄露。

解决方法很简单，在  
`GifFile->Image.ColorMap = NULL;`  
之前加一句  
`GifFreeMapObject(GifFile->Image.ColorMap);`  
即可。

我喜欢VC的原因之一，就是VC有内存泄漏报告机制，即在debug状态下应用退出后，会逐条报告所有未被释放的内存及该条内存申请时的编号，有了编号再找导致内存泄漏的源代码就相对比较容易，下条件断点即可。

（全文完）