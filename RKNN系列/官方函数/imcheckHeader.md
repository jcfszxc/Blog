
```markdown
# imcheckHeader 函数
## 功能
校验当前使用的头文件版本与librga库版本的差异。

## 语法
```cpp
IM_API IM_STATUS imcheckHeader(im_api_version_t header_version = RGA_CURRENT_API_HEADER_VERSION);
```

## 参数
- `header_version`: 头文件版本（默认值为 RGA_CURRENT_API_HEADER_VERSION）

## 描述
这个函数用于检查当前使用的RGA头文件版本是否与librga库版本兼容。它接受一个可选的`im_api_version_t`类型参数来指定头文件版本,默认使用`RGA_CURRENT_API_HEADER_VERSION`宏定义的当前头文件版本。

## 返回值
返回一个`IM_STATUS`枚举值:
- `IM_STATUS_SUCCESS`: 版本校验成功,头文件版本与librga库版本兼容。
- 负值: 版本校验失败,表示不兼容或其他错误。

## 注意事项
### 使用说明
- **头文件引用**
    - C++调用im2d api：需要包含 `im2d.hpp`
    - C调用im2d api：需要包含 `im2d.h`
- **库文件**
    - 需要链接 `librga.so` 或 `librga.a`
- 建议在使用任何其他RGA功能之前先调用此函数进行版本校验。
- 通常使用`RGA_CURRENT_API_HEADER_VERSION`宏作为参数,无需手动指定版本号。

## 示例用法
```cpp
#include <iostream>
#include "im2d.hpp"

int main() {
    IM_STATUS status = imcheckHeader(RGA_CURRENT_API_HEADER_VERSION);
    if (status != IM_STATUS_SUCCESS) {
        std::cerr << "RGA header version check failed. Error code: " << status << std::endl;
        return -1;
    }
    std::cout << "RGA header version check passed successfully." << std::endl;
    // 继续使用其他RGA功能...
    return 0;
}
```

## 示例输出
成功时:
```
RGA header version check passed successfully.
```
失败时:
```
RGA header version check failed. Error code: -1
```

## 依赖
- RGA 库
- C++ 标准库（用于错误输出）
```
这个描述提供了`imcheckHeader`函数的基本信息、用途、参数说明、返回值、使用注意事项和示例用法。
```
