- proj2-gitlet 的先导, 介绍一些持久化, 序列化的方法
- Java 编译, 命令行程序
- 在 java 中与计算机文件系统进行交互
- 在 java 中对一个对象进行序列化和反序列化


# spec
### persistence
- 利用计算机的文件系统进行持久化, 在退出程序时将程序状态作为文件保存在计算机的文件系统中, 在重新进入程序时, 将文件读进程序从而恢复状态


### java and compilation
- `javac`, java 编译器, 将 `.java` 源文件编译成 `.class` 的字节码
- `java`, java 解释器, 将 `.class` 的字节码解释称机器码, 让程序可以在硬件上运行
- `javac *.java`
- `java capers.Main`, 如果主类在某一个 `package` 中, 需要使用完整的主类位置来执行
- `java capers.Main story "this is a single argument"`, 这样子执行的话, `"story", "this is..."` 就会作为 `String[] args` 传递给 Main, 从而实现通过命令行与程序进行交互
- `make`, 由于这个程序涉及到了状态持久化, 不太好在 `java, JUnit` 中测试, 因此使用 make+python 脚本进行测试


### files and directories in java
- `pwd`, 当前所在的目录, 对应到 java 中, 就是当前 java 文件所在的目录, 使用 `System.getProperty("user.dir")` 获得
```java
class Example {  
    public static void main(String[] args) {  
        System.out.println("pwd: " + System.getProperty("user.dir"));  
    }  
}

example $ java Example
pwd: F:\workspace\courses\cs61b\lab6\example
```

- 相对路径: 就是文件到 `pwd` 的路径, 例如上面的 `./example`
- 绝对路径: 就是文件到根目录的路径, 例如上面的 `F:\workspace\courses\cs61b\lab6\example`

- 文件交互, java 内置了一个类 `File`, 允许使用这个类对操作系统文件系统中的目录或文件进行操作
- 一般通过相对路径, 操作当前程序工作目录下的文件
```java
File f = new File("dummy.txt");

// actually create a file in file system
f.createNewFile();

f.exists();

Utils.writeContents(f, "Hello World");

File dir = new File("dummy");
dir.mkdir();
```


### serializable
- 就是将一个 `java object` 进行字节编码, 然后通过字节编码读回原来的对象
- java 中有一个内置的 `Serializable` 接口
```java
import java.io.Serializable;

public class Model implements Serializable {
    ...
}

// 序列化
Model m = ....;
File outFile = new File(saveFileName);
try {
    ObjectOutputStream out =
        new ObjectOutputStream(new FileOutputStream(outFile));
    out.writeObject(m);
    out.close();
} catch (IOException excp) {
    ...
}

// 反序列化
Model m;
File inFile = new File(saveFileName);
try {
    ObjectInputStream inp =
        new ObjectInputStream(new FileInputStream(inFile));
    m = (Model) inp.readObject();
    inp.close();
} catch (IOException | ClassNotFoundException excp) {
    ...
    m = null;
}
```


### Mandatory Epilogue: Debugging
- 这里使用 `make check` 进行测试, 因此似乎不能直接在 idea 中 debug
- 所以使用链接远程 jvm 模式进行 debug
- `python runner.py --keep --debug our/test02-two-part-story.in`
- 然后在 idea 中打好断点, 运行, 等待链接



# code
done
