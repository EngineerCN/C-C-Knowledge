# 强引用导致双重delete情况（double delete）

---

## 错误用法

文件：`shared_ptr_error.cpp`

```cpp
#include <memory>

class Node {

public:
    std::shared_ptr<Node> get_self(){
        // 这处为 {} 函数体，由于退出函数体，局部变量会除非析构，所以此处触发了一次delete
        return std::shared_ptr<Node>(this);
    }

};

int main(){

    // 此时强引用为 1
    std::shared_ptr<Node> ptr1(new Node); 

    // 由于class的this导致指向同一段内存，但强引用仍是1，而不是遇期的2
    std::shared_ptr<Node> ptr2 = ptr1->get_self();

    // 由于强引用是 1
    // return std::shared_ptr<Node>(this); 触发了一次delete，而 ptr2 此时就会触发双重delete
    return 0;
}

```

## 正确用法

文件：`shared_ptr_success.cpp`

```cpp

// 参考文章：https://stackoverflow.com/questions/712279/what-is-the-usefulness-of-enable-shared-from-this

#include <memory>

// C++ 11 提供更为安全的模板，将this指针，转成智能指针
class Node : public std::enable_shared_from_this<Node>{

public:
    std::shared_ptr<Node> get_self(){
        return shared_from_this();
    }

};

int main(){

    std::shared_ptr<Node> ptr1(new Node);
    std::shared_ptr<Node> ptr2 = ptr1->get_self();

    return 0;
}
```

## cmake工程文件

文件：`CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.22)

# 编译环境 gcc11
project(shared_ptr_demo)

set(CMAKE_CXX_STANDARD 11)

add_executable(shared_ptr_error shared_ptr_error.cpp)

add_executable(shared_ptr_success shared_ptr_success.cpp)
```