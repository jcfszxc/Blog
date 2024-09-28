
```markdown
# querystring 函数
## 功能
查询获取当前芯片平台RGA硬件版本与功能支持信息，以字符串的形式返回。

## 语法
```cpp
std::string querystring(int query_type);
```

## 参数
- `query_type`: 要查询的 RGA 信息类型（整数）

## 描述
这个函数用于获取特定类型的 RGA 信息。它接受一个整数参数来指定查询类型，并返回一个包含相应 RGA 信息的字符串。

可能的 `query_type` 值及其含义：
- 0: RGA 厂商信息
- 1: RGA 版本信息
- 2: RGA 支持的最大输入分辨率
- 3: RGA 支持的最大输出分辨率
- 4: RGA 支持的stride对齐要求
- 5: 支持的缩放倍数
- 6: RGA 支持的输入格式
- 7: RGA 支持的输出格式
- 8: RGA 特性
- 9: RGA 预期性能
- 10: 输出所有 RGA 信息

## 返回值
返回一个 `std::string`，包含请求的 RGA 信息。如果查询类型无效，将返回错误信息。

## 注意事项
### 使用说明

- **头文件引用**
    - C++调用im2d api：需要包含 `im2d.hpp`
    - C调用im2d api：需要包含 `im2d.h`
- **库文件**
    - 需要链接 `librga.so` 或 `librga.a`
- 确保在调用此函数之前已正确初始化 RGA 环境。
- 返回的字符串格式可能因查询类型而异。
## 示例用法
```cpp
#include <iostream>
#include "im2d.hpp"

int main() {
    // 查询 RGA 版本信息
    std::string version_info = querystring(1);
    std::cout << version_info << std::endl;

    // 查询所有 RGA 信息
    std::string all_info = querystring(10);
    std::cout << all_info << std::endl;

    return 0;
}
```

## 示例输出

```cpp
rga_api version 1.10.0_[8]
RGA_api version       : v1.10.0_[8]
RGA version           : RGA_2_Enhance RGA_3

RGA vendor            : Rockchip Electronics Co.,Ltd.
RGA_api version       : v1.10.0_[8]
RGA version           : RGA_2_Enhance RGA_3
Max input             : 8192x8192
Max output            : 8128x8128
Byte stride           : 16 byte
Scale limit           : 0.0625 ~ 16
Input support format  : RGBA_8888 RGB_888 RGB_565 RGBA_4444 RGBA_5551 YUV420_sp_8bit YUV420_sp_10bit YUV420_p_8bit YUV420_p_10bit YUV422_sp_8bit YUV422_sp_10bit YUV422_p_8bit YUV422_p_10bit YUYV422 YUV400/Y4
output support format : RGBA_8888 RGB_888 RGB_565 RGBA_4444 RGBA_5551 YUV420_sp_8bit YUV420_sp_10bit YUV420_p_8bit YUV422_sp_8bit YUV422_sp_10bit YUV422_p_8bit YUYV420 YUYV422 YUV400/Y4
RGA feature           : color_fill color_palette ROP quantize src1_r2y_csc dst_full_csc FBC_mode blend_in_YUV BT.2020
expected performance  : max 4 pixel/cycle
```

## 依赖
- RGA 库
- C++ 标准库（用于 std::string）
```
这个说明提供了 `querystring` 函数的基本信息、用途、参数说明、返回值、使用注意事项和示例用法。
```

