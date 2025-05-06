### 如何在rust里面调用c/c++的API呢？---> FFI
***
#### 你的项目结构默认如下
### project
***
- ctest //你的c/c++的API目录
1.  c_src  //你的c/cpp文件
2.  c_include //你的.h/hpp文件
***
- src
  1. main.rs
- cargo.toml
- cargo.lock
- build.rs //必须包含
***
### 一 build.rs
```cpp
// build.rs
fn main() {
    // 编译 C 代码为静态库
    let c_src = "ctest/test.c";
    let c_object = "ctest/test.o";
    let static_lib = "ctest/libtest.a";
    std::process::Command::new("gcc")
        .arg("-c")
        .arg(c_src)
        .arg("-o")
        .arg(c_object)
        .status()
        .unwrap();
    std::process::Command::new("ar")
        .arg("rcs")          // rcs 选项：创建并静态链接
        .arg(static_lib)     // 静态库路径（ctest/libtest.a）
        .arg(c_object)       // 添加目标文件
        .status()
        .expect("Failed to create static library");
    // 告诉 Rust 链接该目标文件
    println!("cargo:rustc-link-search=./ctest");
    println!("cargo:rustc-link-lib=static=test");  // 静态链接
}
```
### 二 main.cpp
 在你的main.cpp文件里面包含，类似如下，**注意**，用rust的方式来声明，对！
 ```cpp
unsafe  extern  "C" {
    unsafe  fn sum(a:i32,b:i32)->i32;
}
...
```
### rust里面使用cpp的API，如class,模板...
1. - 对 std::shared_ptr 使用 cxx::SharedPtr 类型包装，利用 Rust 的 Drop trait 自动释放内存。
***
```cpp
#[cxx::bridge]
mod ffi {
    unsafe extern "C++" {
        include!("cpp_header.h");
        type MyCppClass;
        fn create_shared() -> UniquePtr<MyCppClass>;
    }
}
```
***
2. - 裸指针的 unsafe 封装
对 new/delete 操作需封装为 Rust 的 Box 或自定义智能指针，避免悬垂指针。使用 #[repr(C)] 确保结构体内存布局与 C++ 兼容。
***
3. - 异常安全边界
​零异常契约
强制 C++ 代码通过 noexcept 或返回 std::expected<T, E> 避免异常外泄。
Rust 端通过 try! 宏或 ? 操作符处理 Result。
​SEH 与 Rust 的兼容性
Windows 平台下，使用 #[cfg(windows)] 和 SetUnhandledExceptionFilter 捕获崩溃。
***
4. - Rust 绑定实现步骤：
​定义不透明句柄：使用 #[repr(C)] 结构体表示类实例指针。
​封装构造函数/析构函数：通过外部函数创建和销毁对象。
​方法绑定：将成员函数映射为 Rust 函数，接收句柄作为第一个参数。
***
cpp 代码实现
```cpp
// my_class.h
class MyClass {
public:
    MyClass(int value);
    ~MyClass();
    int get_value() const;
    void set_value(int value);
private:
    int value_;
};
```
***
- rust 代码定义
```cpp
// src/lib.rs
#[repr(C)]
pub struct MyClass(*mut std::ffi::c_void);

extern "C" {
    fn MyClass_new(value: i32) -> MyClass;
    fn MyClass_delete(obj: MyClass);
    fn MyClass_get_value(obj: MyClass) -> i32;
    fn MyClass_set_value(obj: MyClass, value: i32);
}

impl MyClass {
    pub fn new(value: i32) -> Self {
        Self(unsafe { MyClass_new(value) })
    }

    pub fn get_value(&self) -> i32 {
        unsafe { MyClass_get_value(self.0) }
    }

    pub fn set_value(&self, value: i32) {
        unsafe { MyClass_set_value(self.0, value) }
    }
}

// 确保析构函数被调用
impl Drop for MyClass {
    fn drop(&mut self) {
        unsafe { MyClass_delete(self.0) }
    }
}
```
***
### 提醒
- ​优先使用 cxx crate：自动化生成类型安全的 FFI 绑定，减少手动 unsafe 代码。
​隔离不安全代码：将 C++ 交互封装在独立模块，通过 Rust 的 unsafe 块最小化暴露。
​持续验证 ABI：利用 CI 对 Rust 和 C++ 编译器版本组合进行交叉验证。
所以面对大量的cpp新特性。在rust里面调用c++的API最好使用c风格的代码.
***
```cpp
***
###  三 终端执行 
```cpp
1. cargo build
2. cargo run 
