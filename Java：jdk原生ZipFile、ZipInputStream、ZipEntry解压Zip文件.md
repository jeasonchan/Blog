
```java

import lombok.AllArgsConstructor;
import lombok.Getter;
import lombok.Setter;

import java.io.*;
import java.nio.charset.StandardCharsets;
import java.util.ArrayList;
import java.util.List;
import java.util.zip.ZipEntry;
import java.util.zip.ZipFile;
import java.util.zip.ZipInputStream;

@Setter
@Getter
@AllArgsConstructor
public class UnCompressTask {
    private String zipFilePath;
    private String targetDirPAth;

    /**
     * 注意流的关闭！！！！！！！！！！！！！！
     * 此处为了方便，此处并没有在finally中关闭流
     *
     * @return 返回文件名列表
     */
    public List<String> execute() throws IOException {
        List<String> fileNameList = new ArrayList<>();//用于输出解压后文件的路径

        FileInputStream fileInputStream = new FileInputStream(zipFilePath);
        ZipInputStream zipInputStream = new ZipInputStream(fileInputStream, StandardCharsets.UTF_8);//用于获取zipEntry
        ZipFile zipFile = new ZipFile(zipFilePath);//必备，用于获取其中的zipEntry对应的输入流

        ZipEntry zipEntry = null;
        while (null != (zipEntry = zipInputStream.getNextEntry())) {
            String extraFilePath = targetDirPAth + zipEntry.getName();
            fileNameList.add(extraFilePath);

            File file = new File(extraFilePath);
            FileOutputStream fileOutputStream = new FileOutputStream(file);

            //此处未使用缓冲流，大文件时要使用连续的、大片的内存块，会比较慢
            InputStream extraBytesInputStream = zipFile.getInputStream(zipEntry);
            byte[] extraBytes = new byte[extraBytesInputStream.available()];

            //inputstream型的流，只有read方法，将字节从读取出来写入byte[]中
            extraBytesInputStream.read(extraBytes);//一次性读取全部的字节

            //outputstream型的流，只有write方法，从字节数组读取数据，写入到绑定的File对象中
            fileOutputStream.write(extraBytes);
            fileOutputStream.flush();//确保管道（缓存）中没有剩余的字节数据

        }


        return fileNameList;

    }


    /**
     * 写入到文件时，使用了缓冲流
     * <p>
     * 注意Finally 中 IO流的关闭！！！！！！！！！
     *
     * @return
     */
    public List<String> executeWithBuffer() throws IOException {
        List<String> filePathList = new ArrayList<>();

        FileInputStream fileInputStream = new FileInputStream(this.zipFilePath);
        ZipInputStream zipInputStream = new ZipInputStream(fileInputStream);
        ZipFile zipFile = new ZipFile(this.zipFilePath);
        FileOutputStream fileOutputStream = null;
        InputStream inputStream = null;
        BufferedInputStream bufferedInputStream = null;

        ZipEntry zipEntry = null;

        //开始针压缩包内的每个文件进行解压处理
        while (null != (zipEntry = zipInputStream.getNextEntry())) {
            String extraFilePath = this.targetDirPAth + zipEntry.getName();
            filePathList.add(extraFilePath);

            File extraFile = new File(extraFilePath);
            fileOutputStream = new FileOutputStream(extraFile);

            inputStream = zipFile.getInputStream(zipEntry);
            bufferedInputStream = new BufferedInputStream(inputStream);

            byte[] buffer = new byte[1024 * 4];
            int readLength = 0;
            while ((readLength = (bufferedInputStream.read(buffer))) != -1) {
                fileOutputStream.write(buffer, 0, readLength);//每次只读取相应长度的
                fileOutputStream.flush();
            }


        }


        return filePathList;
    }


}
```
