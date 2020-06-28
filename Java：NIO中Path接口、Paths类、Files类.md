# 1 前言
参考文档：
NIO学习：Paths和Files工具类的使用      https://www.cnblogs.com/zxfei/p/10901364.html

# 2 代码实践
自我代码实践

```java
package default_package.NIO练习.Path和Files类练习;

import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;

public class Main {
    public static void main(String[] args) throws IOException {
        // nio中提供的新的文件读写核心：
        // Path接口，Paths类，Files类

        Path textPath = Paths.get("D:\\Users\\10256028\\projects\\personal_exercise\\exerciese_in_zte\\test.txt");

        boolean isExist;

        System.out.println(textPath + " exist: " + (isExist = Files.exists(textPath)));

        if (!isExist) {
            // 父目录必须在，才能创建文件！！！
            Path parentPath;

            if (null != (parentPath = textPath.getParent())) {
                Files.createDirectories(parentPath);

            }

            Files.createFile(textPath);
        }

        //和File一样，只能删除的空文件夹
        //Files.delete();

        //移动文件
//        Path sourcePath = Paths.get("./233333333/1.txt");
//        Path targetPath = Paths.get("./666666/1.txt");
//        Files.move(sourcePath, targetPath);


        //复制文件
        //目标Path指向的一个文件，跟bash里的cp不太一样，cp里面的目标是一个文件夹
//        Files.copy(textPath,Paths.get("23231213/2.txt"));



    }


}
```

参考文章中的代码实践：

```java
package default_package.NIO练习.Path和Files类练习;

import java.io.File;
import java.io.IOException;
import java.nio.file.FileSystems;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardCopyOption;
import java.nio.file.StandardOpenOption;


public class PathFilesDemo {
    public static void main(String[] args) {
        createFileOrDir();
    }

    // 创建文件或目录
    private static void createFileOrDir() {
        try {
            // 创建新目录，除了最后一个部件，其他必须是存在的
            Files.createDirectory(Paths.get("F:/test"));

            // 创建路径中的中间目录，能创建不存在的中间部件
            Files.createDirectories(Paths.get("F:/test/test"));

            // 创建文件
            Files.createFile(Paths.get("F:/testbak.txt"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }// createFileOrDir

    // 删除文件
    private static void deleteFile() {
        Path p = Paths.get("F:/test.txt");
        try {
            Files.delete(p);// 用static boolean deleteIfExists(Path path)方法比较好
            System.out.println("删除成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }// deleteFile

    // 移动文件
    private static void moveFile() {
        Path pSrc = Paths.get("F:/test.txt");
        Path pDest = Paths.get("E:/test.txt");
        try {
            Files.move(pSrc, pDest, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("移动成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }// moveFile

    // 复制文件
    private static void copyFile() {
        Path pSrc = Paths.get("F:/test.txt");
        Path pDest = Paths.get("F:/testbak.txt");
        try {
            Files.copy(pSrc, pDest, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("复制成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }// copyFile

    // 从文件读取数据
    private static void readFromFile() {
        Path p = Paths.get("F:/", "test.txt");
        try {
            byte[] bytes = Files.readAllBytes(p);
            System.out.println(new String(bytes));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    // 向文件写入数据
    private static void write2File() {
        // 获得路径
        Path p = Paths.get("F:/", "test.txt");
        String info = "I love java really,你喜欢什么？";
        try {
            // 向文件中写入信息
            Files.write(p, info.getBytes("utf8"), StandardOpenOption.APPEND);
            System.out.println("写入成功");
        } catch (IOException e) {
            e.printStackTrace();
        }
    }// write2File

    // 获得文件路径的几种方法
    private static void getFilePath() {
        File file = new File("F:/test.txt");
        // Path
        Path p1 = Paths.get("F:/", "test.txt");// F:\test.txt
        System.out.println(p1);

        Path p2 = file.toPath();
        System.out.println(p2);

        Path p3 = FileSystems.getDefault().getPath("F:/", "test.txt");
        System.out.println(p3);
    }// getFilePath
}
```
