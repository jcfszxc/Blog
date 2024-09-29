# RGA 图像缓冲区预处理

## importbuffer_T 接口

对于需要RGA处理的外部内存,可以使用importbuffer_T接口将缓冲区对应的物理地址信息映射到RGA驱动内部,并获取缓冲区相应的地址信息,方便后续稳定、快速地调用RGA完成工作。

### 参数说明

| 参数类型(T)              | 数据类型            | 描述                                   |
| -------------------- | --------------- | ------------------------------------ |
| virtual address      | void *          | 图像缓冲区虚拟地址                            |
| physical address     | uint64_t        | 图像缓冲区连续的物理地址                         |
| fd                   | int             | 图像缓冲区DMA的文件描述符                       |
| GraphicBuffer handle | buffer_handle_t | 图像缓冲区handle, 包含缓冲区地址,文件描述符,分辨率及格式等信息 |
| GraphicBuffer        | GraphicBuffer   | Android graphic buffer               |
| AHardwareBuffer      | AHardwareBuffer | 系统中可被各种硬件组件访问的内存块                    |

> 注意: 不同的buffer类型调用RGA的性能是不同的,性能排序如下:
> 
> physical address > fd > virtual address
> 
> 一般推荐使用fd作为buffer类型。

### 接口定义

#### 基于内存大小的导入

```cpp
IM_API rga_buffer_handle_t importbuffer_fd(int fd, int size);
IM_API rga_buffer_handle_t importbuffer_virtualaddr(void *va, int size);
IM_API rga_buffer_handle_t importbuffer_physicaladdr(uint64_t pa, int size);
```

参数说明:
- fd/va/pa: [必需] 外部缓冲区
- size: [必需] 内存大小

#### 基于图像参数的导入

```cpp
IM_API rga_buffer_handle_t importbuffer_fd(int fd, int width, int height, int format);
IM_API rga_buffer_handle_t importbuffer_virtualaddr(void *va, int width, int height, int format);
IM_API rga_buffer_handle_t importbuffer_physicaladdr(uint64_t pa, int width, int height, int format);
```

参数说明:
- fd/va/pa: [必需] 外部缓冲区
- width: [必需] 图像缓冲区的像素宽度步长
- height: [必需] 图像缓冲区的像素高度步长
- format: [必需] 图像缓冲区的像素格式

#### 基于自定义参数的导入

```cpp
IM_API rga_buffer_handle_t importbuffer_fd(int fd, im_handle_param_t *param);
IM_API rga_buffer_handle_t importbuffer_virtualaddr(void *va, im_handle_param_t *param);
IM_API rga_buffer_handle_t importbuffer_physicaladdr(uint64_t pa, im_handle_param_t *param);
```

参数说明:
- fd/va/pa: [必需] 外部缓冲区
- param: [必需] 配置缓冲区参数

#### 特定buffer类型的导入

```cpp
IM_API rga_buffer_handle_t importbuffer_GraphicBuffer_handle(buffer_handle_t hnd);
IM_API rga_buffer_handle_t importbuffer_GraphicBuffer(sp<GraphicBuffer> buf);
IM_API rga_buffer_handle_t importbuffer_AHardwareBuffer(AHardwareBuffer *buf);
```

参数说明:
- hnd/buf: [必需] 外部缓冲区

### 返回值

所有importbuffer_T函数返回rga_buffer_handle_t类型的值,用于描述内存句柄。

## 使用建议

1. 选择合适的buffer类型可以显著影响性能。建议优先考虑使用physical address,其次是fd,最后是virtual address。

2. 在实际应用中,通常推荐使用fd作为buffer类型,因为它在性能和易用性之间取得了良好的平衡。

3. 根据您的具体需求选择合适的importbuffer_T函数。如果您只知道内存大小,使用第一组函数;如果您有完整的图像参数,使用第二组函数;如果您需要更灵活的参数控制,使用第三组函数。

4. 对于特定的buffer类型(如GraphicBuffer或AHardwareBuffer),使用专门的导入函数可以简化操作并提高效率。

5. 请确保在使用完毕后正确释放导入的buffer,以避免内存泄漏。