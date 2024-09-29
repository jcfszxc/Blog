### importbuffer_fd 

## 功能
将外部内存缓冲区导入到 RGA（Rockchip Graphics Acceleration）驱动中，以便后续使用 RGA 进行图像处理。

## 语法
```cpp
IM_API rga_buffer_handle_t importbuffer_fd(int fd, int size);
IM_API rga_buffer_handle_t importbuffer_fd(int fd, int width, int height, int format);
IM_API rga_buffer_handle_t importbuffer_fd(int fd, im_handle_param_t *param);
```

## 参数
### 版本 1
- `fd`: 图像缓冲区的文件描述符
- `size`: 缓冲区大小（字节）

### 版本 2
- `fd`: 图像缓冲区的文件描述符
- `width`: 图像宽度（像素）
- `height`: 图像高度（像素）
- `format`: 图像格式（使用 RGA 定义的格式，如 RK_FORMAT_RGBA_8888）

### 版本 3
- `fd`: 图像缓冲区的文件描述符
- `param`: 指向 `im_handle_param_t` 结构体的指针，包含详细的缓冲区参数

## 描述
这个函数用于将外部分配的内存缓冲区（通过文件描述符表示）导入到 RGA 驱动中。它允许 RGA 访问和处理这些缓冲区中的图像数据。函数有三个重载版本，提供不同级别的参数控制。

## 返回值
- 成功时返回非零的 `rga_buffer_handle_t` 值，表示导入的缓冲区句柄
- 失败时返回 0

## 注意事项
### 使用说明
- 在使用此函数之前，确保已包含必要的头文件（如 `im2d.h`）
- 选择适当的函数版本取决于可用的信息和所需的控制级别
- 使用完毕后，应当使用 `releasebuffer_handle` 函数来释放导入的缓冲区句柄

## 示例用法
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

    // 使用第一个版本的 importbuffer_fd
    rga_buffer_handle_t handle1 = importbuffer_fd(fd, size);
    if (handle1 == 0) {
        std::cerr << "Failed to import buffer (version 1)" << std::endl;
        return -1;
    }

    // 使用第二个版本的 importbuffer_fd
    rga_buffer_handle_t handle2 = importbuffer_fd(fd, width, height, format);
    if (handle2 == 0) {
        std::cerr << "Failed to import buffer (version 2)" << std::endl;
        return -1;
    }

    // 使用第三个版本的 importbuffer_fd
    im_handle_param_t param;
    memset(&param, 0, sizeof(im_handle_param_t));
    param.width = width;
    param.height = height;
    param.format = format;
    rga_buffer_handle_t handle3 = importbuffer_fd(fd, &param);
    if (handle3 == 0) {
        std::cerr << "Failed to import buffer (version 3)" << std::endl;
        return -1;
    }

    std::cout << "All versions of importbuffer_fd succeeded" << std::endl;

    // 清理资源
    releasebuffer_handle(handle1);
    releasebuffer_handle(handle2);
    releasebuffer_handle(handle3);
    drm_buf_destroy(fd, handle, buf, size);

    return 0;
}
```

## 示例输出
成功时:
```
pagesize is 4096
create width=1280, height=720, bpp=32, size=3686400 dumb buffer
out handle= 1
rga_api version 1.10.0_[8]
All three methods of importbuffer_fd succeeded
```
失败时:
```
Failed to import buffer (version X)
```

## 依赖
- RGA (Rockchip Graphics Acceleration) 库
- DRM (Direct Rendering Manager) 库（用于示例中的缓冲区分配）
- C++ 标准库（用于输出）

## 相关函数
- `releasebuffer_handle`: 用于释放通过 `importbuffer_fd` 导入的缓冲区句柄
- `wrapbuffer_handle`: 用于将导入的缓冲区句柄包装成 RGA 可以使用的 `rga_buffer_t` 结构
- `get_bpp_from_format`: 用于从 RGA 格式获取每像素位数（用于 DRM 缓冲区分配）