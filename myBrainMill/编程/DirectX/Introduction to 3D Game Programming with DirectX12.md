msvc程序
c++的language中，conformance Mode不能使用YES，否则编译不过

书摘
# 4 初始化
## 4.1 预备知识
### 4.1.2 COM组件对象模型
原本COM对象是个接口，但我们把它看作一个c++类。
要获取指向某COM接口的指针，需要借助特定函数或另一个COM接口的函数，不能直接new一个
com对象会统计其引用次数，用完某接口后，调用其Release方法
Microsoft::WRL::ComPtr类可以认为是管理COM对象的智能指针，一个ComPtr实例超过作用域范围后，被自动清除。
常用的三个ComPtr类函数
- Get，返回一个底层指针。
- GetAddressOf，返回指向底层接口指针的地址
- Reset，把管理的指针置为nullptr
### 4.1.4 交换链swapchain
前后台缓冲的这种互换操作称为**呈现presenting**，前台缓冲和后台缓冲被称为**交换链swapchain**
### 4.1.5 深度缓冲
虽然swapchain里面有一前一后两个缓冲区，但是同时只有一个缓冲区用于绘制，因此只需要一个深度缓冲区。
深度缓冲区与模板stencil缓冲区必须同时出现。
### 4.1.6 资源与描述符descriptor

资源，就是某块显存。 ^416resource

纹理，顶点被存放在显存中，需要读
后台缓冲区，深度模板缓冲区放在显存中，需要写
发出绘制命令前，需要把本次绘制使用的资源**绑定bind**到流水线上。但是[[#^416resource|资源]]不是直接与流水线绑定的。
通过一个中间层**描述符descriptor**来告诉渲染流水线这些[[#^416resource|资源]]的信息，即**描述符descriptor**包含了对送往GPU的[[#^416resource|资源]]进行描述的轻量级结构。
一块[[#^416resource|资源]]可以被多个**描述符descriptor**同时描述，因为这块[[#^416resource|资源]]可能在流水线中多次使用，比如先渲染到纹理，再把这块纹理用于贴图。
**视图view**与**描述符descriptor**可以认为是相同概念。如renderTargetView是一种描述符descriptor。
渲染流水线在执行命令时需要知道从哪里读，向哪里写
### 4.1.9 功能级别feature level
为了保持程序与硬件兼容，可以查询当前显卡支持的feature level
### 4.1.10 图形基础结构DXGI
DX graphics infrastructure是一种与DX配合使用的API，处理底层任务。例如swapchain接口，切换全屏和窗口模式，枚举显卡，枚举显示器，枚举显示模式，表面格式信息。
一些关键接口
IDXGIFactory，用于创建swapchain
IDXGIAdapter，枚举显卡，
IDXGIOutput，枚举显示器
### 4.1.12 资源驻留
一般情况[[#^416resource|资源]]被创建后就驻留在显存中，销毁时释放显存。程序可以主动管理资源是否保留在显存中
## 4.2 CPU与GPU的交互
### 4.2.1 命令队列与命令列表
DX12只有延迟渲染这种机制，所有命令被提交至**命令列表command list**，等待被执行。
每个GPU至少有一个**命令队列command queue**，命令列表在命令队列中不会立即被执行，而是排队等待。
ID3D12CommandQueue可以调用ExecuteCommandLists把命令列表送入commandQueue，
ID3D12GraphicsCommandList可以添加命令，添加完成后调用Close结束记录命令。
内存管理类ID3D12CommandAllocator，记录在命令列表内的命令，实际上存储在与之关联的命令分配器CommandAllocator上。命令分配器可以保存两大类命令，一类是GPU直接运行的命令，一类是可供重复使用的命令包bundle
一个commandAllocator可以关联多个CommandList，但是只能有一个CommandList在记录命令，
### 4.2.2 CPU与GPU的同步
可以通过Fence，来强制CPU等待，直到GPU完成所有命令的处理。ID3D12Fence可以通过调用Signal方法在CPU端设置Fence的数值，也可以调用ID3D12CommandQueue::Signal方法在GPU端设置Fence的数值。
ID3D12Fence可以在到达某个数值时发出event，CPU可以调用WaitForSingleObject来等待这个event
所以可以构造这个代码片段。
cpu提交一个commandList，并且最后一个command是让GPU端设置一个fence数值a。
接下来cpu为fence创建一个event，fence到达数值a时触发event。
接下来cpu调用WaitForSingleObject来等待这个event。
这样就实现了CPU与GPU的同步
### 4.2.3 资源转换
如何实现对[[#^416resource|资源]]先写后读
可以给[[#^416resource|资源]]设置一个状态，例如写纹理时，状态必须为渲染目标；读纹理时，状态必须为着色器资源状态。
使用transition resource barrier数组，设置[[#^416resource|资源]]当前处于什么状态，要变为什么状态
在commandList中，调用ResourceBarrier适时添加这样的一个transition，就可以起到告诉GPU[[#^416resource|资源]]必须已经处于某种状态，才可以执行后续命令的作用。
### 4.2.4 命令与多线程
commandList必须一个线程一个，不能多个线程共同访问一个commandList
commandAllocator必须一个线程一个，不能多个线程共同访问一个commandAllocator
commandQueue可以多个线程共同访问一个
## 4.3 初始化Direct3D
1. 创建ID3D12Device
2. 创建一个ID3D12Fence对象，并查询描述符大小
3. 检测用户显卡对`4*MSAA`的支持
4. 创建命令队列，命令列表分配器，主命令列表
5. 创建**swapchain交换链**
6. 创建**descriptorHeap描述符堆**
7. 调整后台缓冲区大小，创建renderTargetView
8. 创建深度缓冲及深度视图
9. 设置**viewport视口**和**scissor裁剪矩形**
### 4.3.1 创建ID3D12Device

### 4.3.5 创建swapchain
指定swapchain中缓冲区的数量，宽高，缓冲区的用处，渲染还是什么其他的，窗口还是全屏
### 4.3.6 创建descriptor堆
swapchain里每个缓冲区都有一个RenderTargetView(RTV)，而swapchain里所有缓冲区只有一个depthStencilView(DSV)
### 4.3.7 创建renderTargetView
为了把后台缓冲区绑定到流水线的输出合并阶段，需要为后台缓冲区创建一个descriptor或view，即renderTargetView
### 4.3.8 创建深度/模板缓冲区及其视图
深度/模板缓冲区本质上是个2D纹理。
填写D3D12_RESOURCE_DESC结构体来描述缓冲区资源，再调用ID3D12Device::CreateCommittedResource方法来创建ID3D12Resource对象，这个函数创建一个[[#^416resource|资源]]与一个堆，并把该[[#^416resource|资源]]提交到这个堆中。
GPU的[[#^416resource|资源]]都存在于堆heap中，本质是具有特定属性的GPU显存块。
使用深度/模板缓冲区前，创建对应的深度/模板视图，并绑定到流水线上。CreateDepthStencilView，
### 4.3.9 viewport
viewport的坐标系原点位于缓冲区左上角，x水平向右，y竖直向下，深度范围`[0,1]`
## 4.6 调试
vs集成了一些工具，见https://learn.microsoft.com/zh-cn/previous-versions/hh873126，http://msdn.microsoft.com/en-us/library/hh315751(v=vs.110).aspx
的gpuview
D3D的函数能返回HRESULT错误码
# 5 渲染流水线
## 5.6 顶点着色阶段
### 5.6.2 观察空间
固定在摄像机上的坐标系为左手系，x轴向摄像机的右，y轴向摄像机的上，观看方向为z轴
#### 5.6.3.2 规格化设备坐标
NDC坐标中，x和y的范围均为`[-1,1]`，z的范围是`[n,f]`
#### 5.6.3.5 归一化深度值
投影变换之后，得到投影空间projection space或齐次裁剪空间homogeneous clipping space的坐标，
再经过透视除法，即全部除以wproj，得到NDC坐标，即n被映射为0，f被映射为1
## 5.9 裁剪
视锥体与几何体相交而导致的裁剪，可以使用Sutherland-Hodgman clipping algorithm来计算，并且可以使用投影空间的齐次坐标来进行。前提是需要知道齐次裁剪空间的6个面`x=w,x=-w,y=w,y=-w,z=0,z=w`
## 5.12 输出合并阶段output merge
ps的输出进入此阶段，blending融合操作是在此阶段进行的。
# 6 绘制几何体
## 6.1 顶点与输入布局
使用一个结构体来描述顶点数据，比如结构体里有位置，颜色，法线。
输入布局描述input layout description，用结构体D3D12_INPUT_LAYOUT_DESC来表示，描述结构体的各成员。
c++程序中的结构体与shader中的程序形参，成员的顺序一致。各成员的size，offset等由D3D12_INPUT_ELEMENT_DESC来描述。
比如c++的结构体
```
struct Vertex1
{
XMFLOAT3 Pos;
XMFLOAT4 Color;
XMFLOAT2 Tex0;
XMFLOAT2 Tex1;
}
```
shader的程序形参
```
VertexOut VS(float3 iPos:POSITION,float4 iColor:NORMAL,float2 iTex0:TEXCOORD0,float2 iTex1:TEXCOORD1)
```
描述各成员的D3D12_INPUT_ELEMENT_DESC为
```
D3D12_INPUT_ELEMENT_DESC desc1[]=
{
{"POSOTION",0,DXGI_FORMAT_R32G32B32_FLOAT,0,0,D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA,0},
{"COLOR",0,DXGI_FORMAT_R32G32B32A32_FLOAT,0,12,D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA,0}
{"TEXCOORD",0,DXGI_FORMAT_R32G32_FLOAT,0,24,D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA,0}
{"TEXCOORD",1,DXGI_FORMAT_R32G32_FLOAT,0,32,D3D12_INPUT_CLASSIFICATION_PER_VERTEX_DATA,0}
}
```
描述结构体的D3D12_INPUT_LAYOUT_DESC为，不知道，程序代码里没有这结构体
```

```
## 6.2 VertexBuffer顶点缓冲区
为了使GPU能访问顶点数组，需要把顶点数组放在GPU[[#^416resource|资源]]里，这样的GPU[[#^416resource|资源]]叫顶点缓冲区VertexBuffer。
填写D3D12_RESOURCE_DESC结构体来描述缓冲区资源，再调用ID3D12Device::CreateCommittedResource方法来创建ID3D12Resource对象。
静态几何体一般位于默认堆中，只有GPU可以访问此堆。
如何初始化默认堆的顶点数据？在上传堆中创建一个顶点缓冲区，先把数据从CPU复制到上传堆中的顶点缓冲区，再把顶点数据从上传堆复制到默认堆。
为了使用顶点缓冲区，还要为其创建顶点缓冲区视图vertex buffer view，但是无需为视图创建描述符堆。
然后将视图与流水线上的一个槽slot绑定，这样就能向输入装配器阶段传递顶点数据了ID3D12GraphicsCommandList::IASetVertexBuffers
然后通过调用ID3D12GraphicsCommandList::DrawInstanced真正进行绘制命令
## 6.3 索引index和索引缓冲区
类似顶点缓冲区，创建索引缓冲区视图D3D12_INDEX_BUFFER_VIEW，然后绑定到流水线ID3D12GraphicsCommandList::IASetIndexBuffer
## 6.4 vs
DirectX12中使用HLSL，没有引用和指针的概念。输入输出参数可以写在结构体中。输入参数又叫输入签名input signature。
输入参数怎么与descriptor对应，通过参数后面的语义`:POSITION`
输出参数的语义如果加了`SV_`表示系统值，如`SV_POSITION`存有投影变换后的顶点坐标值。
vs或gs中只能进行投影变换，无法进行透视除法。这是由GPU完成。
## 6.5 fs
fs针对每一个像素片段pixel fragment运行一次。有些pixel fragment最终会成为后台缓冲区的像素，另一些pixel fragment却可能在混合，裁剪，深度处理等阶段被丢弃。
vs的输出与fs的输入，参数数据类型，语义必须匹配
vs的输出可以写为返回值，也可以写为输出参数
fs的输出可以写为返回值，函数要加上`SV_Target`语义，表示输出与[[#4.3.7 创建renderTargetView|renderTargetFormat]]匹配
## 6.6 常量缓冲区constant buffer
常量缓冲区也是一种GPU[[#^416resource|资源]]，数据内容供shader使用。
### 6.6.1 创建
shader中可以这样声明一个常量缓冲区对象
```
cbuffer cbPerObject:register(b0)
{ float posX;};
```
或者
```
struct ObjectConstants
{
float4x4 worldViewProj;
uint id;
};
ConstantBuffer<ObjectConstants> gObjCnst:register(b0);
```
cpu每帧更新一次，所以经常创建到上传堆中
大小必须为硬件最小分配空间有(可能为256byte)的整数倍
### 6.6.2 更新数据
在cpu中把cb通过内存映射到一块内存。然后内存复制进行更新数据。
```
ComPtr<ID3D12Resource> mUploadBuffer;//cb对象
BYTE* mappedPtr=nullptr;//一个内存地址
mUploadBuffer->Map(,,mappedPtr);//cb对象映射到内存地址
memcpy_s(mappedPtr,size,&数据,sizeof(数据));//复制数据
if(mUploadBuffer!=nullptr)
mUploadBuffer->Unmap(0,nullptr);//结束cb对象内存映射
mappedPtr=nullptr;//删除那个内存地址
```
### 6.6.3 上传缓冲区
每帧把数据从内存复制到cb的内存映射时，要与GPU的执行进行同步。
### 6.6.4 cb的descriptor
cb的descriptor需要放在D3D12_DESCRIPTOR_HEAP_TYPE_CBV_SRV_UAV类型的描述符堆里。这种堆可以存放cb，着色器资源描述符shader resource view，无序访问描述符unordered access
创建描述符堆CreateDescriptorHeap，
创建描述符CreateConstantBufferView
### 6.6.5 根签名和描述符表
绘制调用开始执行前，把shader需要的各种资源绑定到流水线，即绑定到特有的寄存器槽register slot上。

寄存器槽就是向shader传递资源的方法，`register(*#)`，`*`表示资源类型，如s(采样器)，u(无序访问)，t(着色器资源)，b(常量缓冲区)，`#`表示寄存器编号。  ^registerslot

在shader中
```
cbuffer cbPass:register(b1)//把cb绑定到cb寄存器槽1
Texture2d gDiffuseMap:register(t0)//把纹理绑定到纹理寄存器槽0
SamplerState gsamPointClamp:register(s1)//把采样器绑定到采样器寄存器槽1
```
根签名root signature，绘制调用开始执行前，流水线要绑定哪些资源，这些资源被映射到哪个[[#^registerslot|寄存器槽]]。  ^rootSignature

## 6.7 编译shader
shader首先被编译为可移植的字节码，接下来，驱动程序将字节码重新编译为当前系统GPU优化的本地指令。
shader可以离线编译为.cso(complied shader object)文件，使用DX自带的FXC命令行编译工具。
可以编译debug版和release版
可以生成汇编代码
然后再用c++的输入binary文件机制读取cso文件，
## 6.8光栅器状态
即不能在shader中指定或实现的功能
fillMode，线框或实体
cullmode，背面或正面，或禁用
winding，ccw或cw
multisample，true或false
antialiasedLine，true或false
## 6.9 流水线状态对象
重要的哲学
D3D实质上就是状态机，ID3D12PipelineState对象集合了大量流水线状态信息。把状态集中在一起，即便于检查这些设置是否兼容，也便于一起将这些状态送到流水线。而且驱动程序可以在初始化时生成对流水线编程的全部代码，减少渲染时的开销。
视口viewport和裁剪矩形scissor等属性不在pso之内，因为可能随时改变。
控制流水线状态的对象被称为**流水线状态对象pipeline state object**
ID3D12PipelineState接口表示D3D12_GRAPHICS_PIPELINE_STATE_DESC结构体描述
    结构体里包含vsfs等待绑定的shader，blendState，多重采样的mask，RasterizerState光栅器状态，DepthStencilState深度模板状态，InputLayout输入布局描述，PrimitiveTopologyType图元拓扑类型，渲染目标的数量，渲染目标的格式，深度模板缓冲区的格式，多重采样描述，
CreateGraphicsPipelineState函数来创建ID3D12PipelineState对象
## 6.10 几何图形辅助结构体
一个结构体，存储了vertexbuffer和indexbuffer的属性，数据，视图
# 13 Compute shader
## 13.1 thread和thread group
线程组成group，group在一个处理器中运行。最好每个处理器中有至少两个group。对于N卡每个group内部的线程数量最好是32的倍数，每32个线程叫做一个warp。
group内部的线程可以共享显存，可以同步。不同group的线程不能共享显存，不能同步。
调用`ID3D12GraphicsCommandList::Dispatch`来启动线程group，参数是个三维向量，每个维度有多少个group。可以把这三个维度理解为一个立方体
## 13.2 compute shader
cs内部写明每个group的线程维度，同样是个三维向量，可以理解为立方体。记得每个group内部的线程数量最好是32的倍数。
cs内部可以通过constant buffer访问全局变量
## 13.3 数据读写
cs可以读写纹理和buffer。
输出数据类型要使有UAV，即unordered access view
cs输出可以调用`ID3D12GraphicsCommandList::CopyResource`复制到内存
## 13.4 线程的id
每个group有自己的id，SV_GroupID
group的线程有自己的相对id，SV_GroupThreadID
每个线程有自己的全局id，SV_DispatchThreadID
## 13.5 
buffer除了固定大小，也可以做成向queue一样的FIFO容器，可以取出，可以加入。但是总的大小受限。
## 13.6 同步和共享内存
每个group最大有32kb的共享内存，可以供内部的thread使用，但是要注意是group内部的所有thread共享这32kb，所以group内部每个线程使用的共享内存是32kb的一部分，如果每个线程使用的共享内存大于平均数，会影响并行。
一个例子是blur算法
一个group内部的所有thread先读出纹理的全部数据，例如每个thread读一列像素，保存在共享内存中，然后进行内存同步，等待所有thread全部读入数据。最后进行blur算法，每个线程访问共享内存，使用自己读入的像素和邻居线程读入的像素进行blur