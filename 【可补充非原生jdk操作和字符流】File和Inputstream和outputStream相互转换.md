@[TOC]
# 1 File转Inputstream
```java
    public static InputStream fileToInputStream(File file) {
        InputStream inputStream=null;
        try {
            inputStream = new FileInputStream(file);

        } catch (FileNotFoundException e) {
            e.printStackTrace();
        }

        return inputStream;
    }
```
很简单，直接用文件输入流，将文件转化为一个输入流即可，这个流是字节流，和文件内部的编码格式无关。
# 2 File转outputStream
# 3 InputStream转File
```java
        try {
            byte[] bytes=new byte[inputStream.available()];
            inputStream.read(bytes);  //将字节流中的字节读取到相同的字节数组中
            String outString=new String(bytes);
            System.out.println(outString);
        } catch (IOException e) {
            e.printStackTrace();
        }
```
# 4 InputStream转outputStream
# 6 ouputStream转File
```java
    public static void stringAddToFile(String extraString, File file) {
        if (extraString == null || file == null) {
            return;
        }
        extraString = "\n" + extraString;//使之后内容以换行“\n”的形式加入到文件中
        byte[] bytes = extraString.getBytes();
        OutputStream outputStream = null;
        try {
            outputStream = new FileOutputStream(file, true); //将文件输出流和将要接收输出流的文件绑定，并且以append的模式写入
            //不写的true就是直接覆盖源文件了
            outputStream.write(bytes);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            closeStream(outputStream);
        }
```
# 7 ouputStream转InputStream
# 8 流程总结
1. inputstream和outputStream的read()和write()方法，其实就是在字节流和字节数组间搬运东西字节，因此，读写方法的入参都是字节数组
2. 文件读取为文本流程：以文件实例为入参，新建文件输入流实例，将文件输入流实例中的字节读取进字节数组，以得到的自己数组为入参新建字符串，即为文件中的字符串，关闭文件输入流，结束。
3. 向文件写入文本流程：以文件实例为入参，新建文件输出流实例（注意是追加模式还是覆盖模式），从字符串getBytes()得到字符串对应的字节数组，再将字节数组中的字节写入到文件输出流中，关闭文件输出流，结束。
