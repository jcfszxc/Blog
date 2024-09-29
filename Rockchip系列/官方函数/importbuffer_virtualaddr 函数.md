### importbuffer_virtualaddr

> 对于需要RGA处理的外部内存，可以使用importbuffer_virtualaddr接口将缓冲区对应的虚拟地址信息导入到RGA驱动内部，并获取缓冲区相应的地址信息，方便后续稳定、快速地调用RGA完成工作。

#### 函数原型

RGA提供了三个版本的 importbuffer_virtualaddr 函数：

```cpp
IM_API rga_buffer_handle_t importbuffer_virtualaddr(void *va, int size);
IM_API rga_buffer_handle_t importbuffer_virtualaddr(void *va, int width, int height, int format);
IM_API rga_buffer_handle_t importbuffer_virtualaddr(void *va, im_handle_param_t *param);
```

#### 参数说明

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| va     | **[必填]** 指向缓冲区的虚拟地址指针                          |
| size   | **[必填]** 缓冲区的大小（以字节为单位）                      |
| width  | **[必填]** 图像宽度（以像素为单位）                          |
| height | **[必填]** 图像高度（以像素为单位）                          |
| format | **[必填]** 图像格式（例如 RK_FORMAT_RGBA_8888）              |
| param  | **[必填]** 指向 im_handle_param_t 结构体的指针，包含更多图像参数 |

#### 返回值

返回 rga_buffer_handle_t 类型的句柄，用于描述导入的内存。如果导入失败，返回 0。

#### 使用说明

1. 第一个版本适用于只知道内存虚拟地址和大小的情况。
2. 第二个版本适用于处理具有已知宽度、高度和格式的图像数据。
3. 第三个版本提供最大的灵活性，允许通过 im_handle_param_t 结构体指定更多参数。

#### 示例代码

```cpp
#include <iostream>
#include "im2d.h"
#include "RgaUtils.h"

int main() {
    int width = 1280;
    int height = 720;
    int format = RK_FORMAT_RGBA_8888;
    size_t size = width * height * get_bpp_from_format(format);

    // 分配内存
    void* buf = malloc(size);
    if (buf == nullptr) {
        std::cerr << "Failed to allocate buffer!" << std::endl;
        return -1;
    }

    // 使用第一个版本
    rga_buffer_handle_t handle1 = importbuffer_virtualaddr(buf, size);

    // 使用第二个版本
    rga_buffer_handle_t handle2 = importbuffer_virtualaddr(buf, width, height, format);

    // 使用第三个版本
    im_handle_param_t param;
    memset(&param, 0, sizeof(im_handle_param_t));
    param.width = width;
    param.height = height;
    param.format = format;
    rga_buffer_handle_t handle3 = importbuffer_virtualaddr(buf, &param);

    // 使用完毕后释放资源
    releasebuffer_handle(handle1);
    releasebuffer_handle(handle2);
    releasebuffer_handle(handle3);
    free(buf);

    return 0;
}
```

#### 注意事项

- 使用 importbuffer_virtualaddr 导入的缓冲区在不再需要时，应使用 releasebuffer_handle 释放。
- 确保传递给函数的参数正确，特别是在使用第二和第三个版本时，图像的宽度、高度和格式必须准确。
- 导入的缓冲区应该是连续的物理内存，以确保 RGA 能够正确访问。