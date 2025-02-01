<div style="display: flex; justify-content: space-evenly; align-items: center; flex-wrap: wrap; width: 100%; overflow: auto; gap: 20px;">
    <img src="https://github.com/pxx917144686/Surge_pxx/blob/main/x/01.png?raw=true" width="300" />
    <img src="https://github.com/pxx917144686/Surge_pxx/blob/main/x/02.png?raw=true" width="300" />
    <img src="https://github.com/pxx917144686/Surge_pxx/blob/main/x/03.png?raw=true" width="300" />
    <img src="https://github.com/pxx917144686/Surge_pxx/blob/main/x/04.png?raw=true" width="300" />
    <img src="https://github.com/pxx917144686/Surge_pxx/blob/main/x/05.png?raw=true" width="300" />
    <img src="https://github.com/pxx917144686/Surge_pxx/blob/main/x/06.png?raw=true" width="300" />
</div>

<table>
<tr>
<td style="padding-right: 20px;">
    <img src="./x/theos.png" width="400" height="180" />
</td>
<td>
    <pre>
    终端执行 克隆 Theos 仓库
    git clone --recursive https://github.com/theos/theos.git

    将 Theos 的路径添加到环境变量中：
    方法一：
    终端执行 直接添加到 ~/theos

    export THEOS=~/theos
    export PATH=$THEOS/bin:$PATH

    终端执行 重新 加载配置：
    source ~/.zshrc

    另一种方法：
    终端执行 打开配置文件 .zshrc
    nano ~/.zshrc

    # Theos 配置  // theos文件夹 的本地路径
    export THEOS=/Users/pxx917144686/theos     

    之后；contron + X 是退出编辑； 按‘y’ 保存编辑退出！

    终端执行 重新 加载配置：
    source ~/.zshrc
    </pre>
</td>
</tr>
</table>

</details>

<!-- Theos 清理、编译、打包 -->
![Preview](./x/编译.png)

<details>
<summary> 👉  如果 theos 报错 </summary>

| **theos报错** | **解释** |
|----------|----------|
| **报错** | ld: warning: -multiply_defined is obsolete |
| **解释** | 为什么会出现这个问题？ |
| **原因** | 新版本的 Apple 链接器 (ld64) 不再推荐使用 `-multiply_defined`；Theos 为了兼容旧版本 iOS，才默认加入该选项。 |
| **解决** | 在文件 `theos/makefiles/targets/_common/darwin_tail.mk` 中找到并删除 `-multiply_defined`。 |

</details>

<details>
<summary> 👉  如果 make 报错 </summary>

| **make报错**                           | **解释**                                                                                                                                                                                                                                                                         |
|------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **报错**                           | `warning: ignoring file '/usr/local/Cellar/openssl@3/3.4.0/lib/libcrypto.3.dylib': found architecture 'x86_64', required architecture 'arm64e'`                                                                                                                                |
| **解释**                           | 英特尔的Mac x86_64 不匹配架构 OpenSSL 库。                                                                                                                                                                                                            |
| **检查**                           | Mac 架构检查：<br>- 在终端执行 `uname -m` <br>- 输出 `x86_64` 表示 Intel Mac<br>- 输出 `arm64` 表示 Apple Silicon Mac                                                                                                                     |
| **解决（Intel x86_64 方法）**         | 避免耽误时间精力！网络指导可能产生误导，把 Intel (x86_64) 的方法误导为适用于 arm64 的方法。使用 Intel Mac 编译 iOS 插件时，目标架构应为 `arm64` 或 `arm64e`。                                                                                                    |
| **步骤一：下载 OpenSSL 官方源代码** | - 在终端执行：`git clone https://github.com/openssl/openssl.git` <br>- 进入目录：`cd openssl`                                                                                                                                                                                   |
| **步骤二：设置环境变量**             | - 执行：`export PLATFORM="iPhoneOS"` <br>- 执行：`export SDK=$(xcrun --sdk iphoneos --show-sdk-path)` <br>- 执行：`export CC="$(xcrun --sdk iphoneos -f clang)"`                                                                                                               |
| **设置支持多个架构**                | - 执行：`export ARCHS="arm64 arm64e"` <br>- 执行：`export CFLAGS="-arch arm64 -arch arm64e -isysroot $SDK -miphoneos-version-min=14.0"` <br>- 执行：`export LDFLAGS="-arch arm64 -arch arm64e -isysroot $SDK"`                                                  |
| **配置 OpenSSL 编译**              | 执行：`./Configure ios64-cross no-shared no-dso no-hw no-engine --prefix=$(PWD)/../openssl-ios`                                                                                                                                                                                  |
| **步骤三：编译 OpenSSL**            | - 清理缓存：`make clean` <br>- 编译 OpenSSL：`make` <br>- 安装 OpenSSL 到指定目录：`make install`                                                                                                                                                                               |
| **验证编译结果**                   | 在终端执行：<br>- `lipo -info ../openssl-ios/lib/libcrypto.a` <br>- `lipo -info ../openssl-ios/lib/libssl.a` <br>成功的输出应显示：`arm64 arm64e`                                                                                                                               |

</details>

<h1 align="center">
  <br>
  关于. `（Tweak）核心代码 `
</h1>

![Preview](./x/Tweak.x.png)

## 目录 - 简单的概括

Objective-C 的运行时编程、动态 Hook（通过 Theos/Logos 语法）、加密算法以及网络请求拦截技术，对目标 iOS 应用进行动态修改，达到反调试、绕过 license 校验和解锁功能的目的。

---

| **模块/方法**                                         | **解释**                                                                                              |
|------------------------------------------------------|-------------------------------------------------------------------------------------------------------|
| **`#import <CoreFoundation/CoreFoundation.h>`**      | 引入 CoreFoundation，提供底层数据结构和内存管理支持。                                                     |
| **`#import <UIKit/UIKit.h>`**                         | 引入 UIKit，用于构建 UI、处理视图与用户交互。                                                             |
| **`#import <objc/runtime.h>`**                        | 导入运行时库，支持动态方法交换、类检测和运行时 Hook 操作。                                                |
| **`%hook UIViewController`**                         | Hook UIViewController 类，通过拦截 `viewDidAppear:` 方法实现自定义弹窗提示。                              |
| **`%hook SGNSARequestHelper`**                       | 拦截关键 API 请求（如 `/api/modules/v2`、`/api/license/verify` 等），返回伪造数据，绕过后端校验。         |
| **`%ctor`**                                          | 构造函数，在动态库加载时执行，用于初始化绕过 OpenSSL 签名验证及其它启动时操作。                           |

---

## 关于 Objective-C 的头文件引用

引用 第三方 头文件，提供 UI 构建到加密算法、网络请求、动态库加载。

---

| **头文件**                                        | **解释**                                                                                             |
|---------------------------------------------------|------------------------------------------------------------------------------------------------------|
| **`#import <CoreFoundation/CoreFoundation.h>`**    | 提供底层数据类型、内存管理及基础服务。                                                                  |
| **`#import <Foundation/Foundation.h>`**            | 提供面向对象的基础类（如 NSString、NSArray 等）。                                                     |
| **`#import <UIKit/UIKit.h>`**                       | 用于 UI 构建、视图管理及事件响应。                                                                     |
| **`#import <CommonCrypto/CommonDigest.h>`**         | 提供 SHA256 等加密摘要算法。                                                                           |
| **`#import <CommonCrypto/CommonCryptor.h>`**        | 提供 AES 加密算法，用于加密 license 数据。                                                              |
| **`#import <objc/runtime.h>`**                      | 支持运行时操作，例如动态方法交换、类检测、修改私有变量。                                                |
| **`#import <dlfcn.h>`**                             | 用于动态加载库及解析符号地址。                                                                          |
| **`#import <mach-o/dyld.h>`**                       | 提供动态模块加载与符号查找功能，便于实现 Hook 操作。                                                    |
| **`#import <mach/mach.h>`**                         | 提供与内核通信的接口。                                                                                |
| **`#import <sys/sysctl.h>`**                        | 用于查询系统状态信息，如检测调试器。                                                                    |
| **`#import <sys/utsname.h>`**                       | 获取系统和硬件信息，用于判断是否在模拟器环境下运行。                                                   |

---

## 关于全局定义与函数声明

一些全局常量、全局变量及辅助函数，用于后续业务逻辑及加密、检测、混淆操作。

---

| **定义/函数**                                                      | **解释**                                                                                           |
|---------------------------------------------------------------------|----------------------------------------------------------------------------------------------------|
| **`#define FIXED_EXPIRATION_DATE 2524608000`**                       | 定义一个固定的过期时间，通常用于伪造 license 信息中的失效日期。                                      |
| **`char LicEncContent[] = "\x03\x04\x02NSExtension";`**              | 定义静态许可加密内容的原始字节数据，用于后续生成 license 加密信息。                                   |
| **`BOOL isDebuggerAttached();`**                                    | 声明检测当前进程是否被调试器附加的函数。                                                              |
| **`BOOL isRunningInSimulator();`**                                  | 声明检测当前运行环境是否为模拟器的函数。                                                              |
| **`BOOL verifyIntegrity();`**                                       | 声明文件完整性校验接口（目前始终返回 YES）。                                                         |
| **`NSString* sha256(NSData *data);`**                                | 声明计算数据 SHA256 摘要并返回十六进制字符串的函数。                                                |
| **`void confuseStaticAnalysis();`**                                 | 声明混淆静态分析的辅助函数，通过伪代码增加逆向工程难度。                                             |
| **`static UIViewController* topViewController(UIViewController *rootVC);`** | 声明递归查找当前最顶层视图控制器的辅助函数，确保 UI 弹窗显示在正确的界面上。                           |

---

## 关于 UIViewController 的 Hook 与弹窗逻辑

对 UIViewController 进行拦截，在视图出现后自动检测环境并展示提示弹窗。

---

| **方法/模块**                                  | **解释**                                                                                          |
|------------------------------------------------|---------------------------------------------------------------------------------------------------|
| **`%hook UIViewController`**                    | Hook UIViewController 类，拦截其 `viewDidAppear:` 方法以添加自定义逻辑。                             |
| **`- (void)viewDidAppear:(BOOL)animated`**       | 在视图显示后调用原始方法，再判断是否满足展示弹窗的条件（非调试器、非模拟器），调用 `showAlert`。      |
| **`- (BOOL)shouldShowAlert`**                    | 判断当前是否允许显示弹窗，避免在调试或模拟器环境下干扰操作。                                         |
| **`- (void)showAlert`**                          | 构造并展示一个 UIAlertController 弹窗，提供“不同意”（退出应用）和“好的”两个选项。                    |
| **`getActiveTopViewController()`**             | 辅助函数，获取当前处于前台的顶层视图控制器，确保弹窗显示在正确的 UI 层级上。                          |
| **`%end`**                                     | 结束 UIViewController 的 Hook 代码块。                                                             |
