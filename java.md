

## 一、快速开始
### 1.下载JDK
JDK下载链接：[JDK](https://www.oracle.com/java/technologies/downloads/#java21)
```
# 验证
java 
javac
java -version
javac -version
```
#### JDK 组成结构
 **JDK (Java Development Kit，Java开发工具包)**
  - **JRE (Java Runtime Environment，Java运行环境)**
    - **JVM (Java Virtual Machine，Java虚拟机)**
    - 核心类库（Java核心API）
  - 开发工具（如 `javac` 编译器、`java` 运行工具等）


### 2. 配置JDK的环境变量
以方便在命令行窗口的任意目录下直接通过命令启动该程序。
```
# 较新的JDK会自动配置
# 较老的JDK，要手动将jdk的bin文件夹的路径复制到path环境变量中
```

### 3. 编写代码并运行
```
# 创建一个名为"HelloWorld.java"的文件，输入以下内容
public class HelloWorld {         # 类名与文件名一致
    public static void main(String[] args) {
        System.out.println("Hello, World!");
    }
}

# 编译代码
javac HelloWorld.java

# 执行代码（不带后缀）
java HelloWorld 

```

project - modules - package - class