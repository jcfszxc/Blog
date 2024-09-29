### releasebuffer_handle

> 当使用外部内存调用RGA完毕后,需要通过内存句柄 handle 调用 releasebuffer_handle 解除该缓冲区与RGA驱动的映射和绑定关系,并释放RGA驱动内部对应的资源。

#### 函数原型

```cpp
IM_API IM_STATUS releasebuffer_handle(rga_buffer_handle_t handle);
```

#### 参数说明

| 参数    | 描述                                                         |
| ------- | ------------------------------------------------------------ |
| handle  | **[必填]** 通过 importbuffer 系列函数获得的缓冲区句柄         |

#### 返回值

返回 IM_STATUS 类型的状态码。成功时返回 IM_STATUS_SUCCESS,失败时返回相应的错误码。

#### 使用说明

1. 在不再需要使用通过 importbuffer 系列函数导入的缓冲区时,应调用此函数释放资源。
2. 释放后的 handle 不应再被使用,否则可能导致未定义的行为。

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

    // 在这里可以使用 rga_handle 进行 RGA 操作
    // ...

    // 操作完成后,释放 RGA handle
    IM_STATUS status = releasebuffer_handle(rga_handle);
    if (status != IM_STATUS_SUCCESS) {
        std::cerr << "Failed to release buffer handle!" << std::endl;
        drm_buf_destroy(fd, handle, buf, size);
        return -1;
    }

    std::cout << "Buffer handle released successfully." << std::endl;

    // 清理 DRM 资源
    drm_buf_destroy(fd, handle, buf, size);

    return 0;
}
```

#### 注意事项

- 每个通过 importbuffer 系列函数获得的 handle 都应该有对应的 releasebuffer_handle 调用。
- 避免重复释放同一个 handle,这可能导致未定义的行为。
- 在多线程环境中使用时,需要确保 releasebuffer_handle 的调用是线程安全的。
- 释放 handle 后,不应再使用该 handle 进行任何操作。
- 如果程序中频繁地导入和释放缓冲区,可能会影响性能。在这种情况下,考虑重用 handle 而不是频繁地导入和释放。