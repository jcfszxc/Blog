### wrapbuffer_handle

> wrapbuffer_handle 函数用于将输入输出的图像参数转化为统一的 rga_buffer_t 结构,作为 IM2D 图形库用户 API 的输入参数。在执行相应的图像操作之前,需要先调用此函数来准备图像数据。

#### 函数原型

```cpp
IM_API rga_buffer_t wrapbuffer_handle(rga_buffer_handle_t handle,
                                      int width,
                                      int height,
                                      int format,
                                      int wstride = width,
                                      int hstride = height);
```

#### 参数说明

| 参数      | 描述                             |
| ------- | ------------------------------ |
| handle  | **[必填]** RGA 缓冲区句柄             |
| width   | **[必填]** 需要处理的图像像素宽度           |
| height  | **[必填]** 需要处理的图像像素高度           |
| format  | **[必填]** 像素格式                  |
| wstride | **[可选]** 图像的像素宽度步幅,默认值为 width  |
| hstride | **[可选]** 图像的像素高度步幅,默认值为 height |

#### 返回值

返回 rga_buffer_t 类型的结构体,用于描述图像信息。

#### 使用说明

1. 在使用 IM2D 图形库的用户 API 之前,需要调用此函数将图像参数转换为统一的格式。
2. 函数返回的 rga_buffer_t 结构体可以直接用作其他 IM2D API 的输入参数。
3. wstride 和 hstride 参数允许处理具有内存对齐或填充的图像。

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
    int flags = 0;

    // 使用 DRM 分配缓冲区
    void* buf = drm_buf_alloc(width, height, get_bpp_from_format(format) * 8, &fd, &handle, &size, flags);
    if (buf == nullptr) {
        std::cerr << "Failed to allocate DRM buffer!" << std::endl;
        return -1;
    }

    // 使用 importbuffer_fd 导入缓冲区
    rga_buffer_handle_t rga_handle = importbuffer_fd(fd, width, height, format);
    if (rga_handle == 0) {
        std::cerr << "Failed to import buffer" << std::endl;
        drm_buf_destroy(fd, handle, buf, size);
        return -1;
    }

    std::cout << "Buffer imported successfully" << std::endl;

    // 使用 wrapbuffer_handle 创建 rga_buffer_t 结构
    rga_buffer_t rga_buffer = wrapbuffer_handle(rga_handle, width, height, format);
    if (rga_buffer.handle == 0) {
        std::cerr << "Failed to wrap buffer" << std::endl;
        releasebuffer_handle(rga_handle);
        drm_buf_destroy(fd, handle, buf, size);
        return -1;
    }

    std::cout << "Buffer wrapped successfully" << std::endl;

    // 这里可以使用 rga_buffer 进行 RGA 操作
    // 例如:
    // IM_STATUS status = imcopy(rga_buffer, rga_buffer);

    // 清理资源
    releasebuffer_handle(rga_handle);
    drm_buf_destroy(fd, handle, buf, size);

    return 0;
}
}
```

这个示例代码展示了如何:

1. 使用 DRM 分配缓冲区
2. 使用 `importbuffer_fd` 导入缓冲区
3. 使用 `wrapbuffer_handle` 创建 `rga_buffer_t` 结构
4. 适当地清理资源

#### 注意事项

- 确保传入的 handle 是有效的 RGA 缓冲区句柄。
- width 和 height 应该与实际图像尺寸相匹配。
- format 参数应该使用 RGA 支持的格式常量。
- 如果图像有特殊的内存布局,可以通过 wstride 和 hstride 参数来指定。
- 在多线程环境中使用时,需要确保 wrapbuffer_handle 的调用是线程安全的。
- 返回的 rga_buffer_t 结构体仅在相关的 RGA 操作期间有效,不应长期保存或在 RGA 上下文之外使用。