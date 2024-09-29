### importbuffer_physicaladdr

> 对于需要RGA处理的外部物理内存，可以使用importbuffer_physicaladdr接口将缓冲区对应的物理地址信息导入到RGA驱动内部，并获取缓冲区相应的地址信息，方便后续稳定、快速地调用RGA完成工作。

#### 函数原型

RGA提供了三个版本的 importbuffer_physicaladdr 函数：

```cpp
IM_API rga_buffer_handle_t importbuffer_physicaladdr(uint64_t pa, int size);
IM_API rga_buffer_handle_t importbuffer_physicaladdr(uint64_t pa, int width, int height, int format);
IM_API rga_buffer_handle_t importbuffer_physicaladdr(uint64_t pa, im_handle_param_t *param);
```

#### 参数说明

| 参数     | 描述                                            |
| ------ | --------------------------------------------- |
| pa     | **[必填]** 缓冲区的物理地址                             |
| size   | **[必填]** 缓冲区的大小（以字节为单位）                       |
| width  | **[必填]** 图像宽度（以像素为单位）                         |
| height | **[必填]** 图像高度（以像素为单位）                         |
| format | **[必填]** 图像格式（例如 RK_FORMAT_RGBA_8888）         |
| param  | **[必填]** 指向 im_handle_param_t 结构体的指针，包含更多图像参数 |

#### 返回值

返回 rga_buffer_handle_t 类型的句柄，用于描述导入的内存。如果导入失败，返回 0。

#### 使用说明

1. 第一个版本适用于只知道内存物理地址和大小的情况。
2. 第二个版本适用于处理具有已知宽度、高度和格式的图像数据。
3. 第三个版本提供最大的灵活性，允许通过 im_handle_param_t 结构体指定更多参数。

#### 示例代码

```cpp
#include <iostream>
#include "im2d.h"
#include "RgaUtils.h"
#include "drm_alloc.h"

int main() {
    int width = 1280;
    int height = 720;
    int format = RK_FORMAT_RGBA_8888;
    int fd, handle;
    size_t size;
    int flags = RGA_UTILS_ROCKCHIP_BO_CONTIG;

    // 使用 DRM 分配缓冲区
    void* buf = drm_buf_alloc(width, height, get_bpp_from_format(format) * 8, &fd, &handle, &size, flags);
    if (buf == nullptr) {
        std::cerr << "Failed to allocate DRM buffer!" << std::endl;
        return -1;
    }

    // 获取物理地址
    uint64_t phy_addr = drm_buf_get_phy(handle);
    if (phy_addr == 0) {
        std::cerr << "Failed to get physical address!" << std::endl;
        drm_buf_destroy(fd, handle, buf, size);
        return -1;
    }

    // 使用第一个版本
    rga_buffer_handle_t handle1 = importbuffer_physicaladdr(phy_addr, size);

    // 使用第二个版本
    rga_buffer_handle_t handle2 = importbuffer_physicaladdr(phy_addr, width, height, format);

    // 使用第三个版本
    im_handle_param_t param;
    memset(&param, 0, sizeof(im_handle_param_t));
    param.width = width;
    param.height = height;
    param.format = format;
    rga_buffer_handle_t handle3 = importbuffer_physicaladdr(phy_addr, &param);

    // 使用完毕后释放资源
    releasebuffer_handle(handle1);
    releasebuffer_handle(handle2);
    releasebuffer_handle(handle3);
    drm_buf_destroy(fd, handle, buf, size);

    return 0;
}
```

#### 注意事项

- 使用 importbuffer_physicaladdr 导入的缓冲区在不再需要时，应使用 releasebuffer_handle 释放。
- 确保传递给函数的参数正确，特别是在使用第二和第三个版本时，图像的宽度、高度和格式必须准确。
- 导入的缓冲区必须是有效的物理内存地址，通常通过特定的内存分配方法（如 DRM）获得。
- 在使用物理地址时，需要特别注意内存管理和访问权限，以避免潜在的系统稳定性问题。

