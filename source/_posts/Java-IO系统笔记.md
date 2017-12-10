
---
title: Java-IO系统笔记
date: 2017-04-03 11:14:30
categories: Java
mathjax: false
tags: [Thinking in Java]
---
<!--more-->
## 写在前面
之前因为忽然需要文件处理，查了下后发现觉得IO类结构不是很清晰。在阅读《Thinking in Java》关于这部分的介绍后，特在此记录。需要简单的文件读写请直接阅读`典型输入使用方式`和`典型输出使用方式`部分。

## 输入
### InputStream类
InputStream是用来表示从不同数据源产生输入的类，它有以下子类：

* ByteArrayInputStream：从内存缓冲区
* StringBufferInputStream：将String转为InputStream
* FileInputStream：文件
* PipedInputStream：管道化
* SequenceInputStream：多个InputStream转为一个InputStream
* FilterInputStream：作为装饰器的接口，为其他InputStream类提供有用功能。有以下子类：
    * DataInputStream：按照可移植方式从流中读取基本类型
    * BufferedInputStream ：缓冲区
    * LineNumberInputStream ：跟踪行号，getLineNumber()等方法
    * PushbackInputStream ：通常作为编译器的扫描器，能将读到的最后一个字符回退T

### InputStreamReader类 适配器
简单地说，InputStream和Reader的不同在于前者是字节，后者是字符操作。因此，字节到字符间的转换需要适配器。子类：

* FileReader
```Java
new FileReader("eaxmple.txt") 
//查看源码，发现其实相当于，使用的是默认encoder
new InputStreamReader(new FileInputStream("eaxmple.txt"))
```

### Reader类
子类：

* FilterReader 
    * PushbackReader
* BufferedReader
    * LineNumberReader

### 典型输入使用方式
字符文本
```Java
File file = new File("example.txt");//最好检验存在
if (file.exists()) {
    try {
        FileInputStream stream = new FileInputStream(file);//InputSteam
        InputStreamReader adapter = new InputStreamReader(stream，"GBK");//适配器
        BufferedReader reader = new BufferedReader(adapter);//Reader
        String tmp = reader.readLine();//读到内存的例子
        reader.flush();//此处可省略
        reader.close();
    } catch (IOException e) {
		e.printStackTrace();
	}
}
```
二进制文件
```Java
File file = new File("example.txt");
if (file.exists()) {
    try {
        FileInputStream stream = new FileInputStream(file);
        BufferedInputStream buffer = new BufferedInputStream(stream);
    } catch (IOException e) {
		e.printStackTrace();
	}
}
```
* 在reader.readLine()等用完后，reader.flush()是将缓冲区的内容进行实际的IO操作，如果不需要多次flush刷新的话，可以省略。因为最后reader.close()时会自动调用flush方法。
* FileInputStream可以传File，也可以传入String来构造。根据源码，传入String后仍旧是new File对象再构造FileInputStream。

## 输出
### OutputStream类
子类：

* ByteArrayOutputStream：放到在内存创建的缓冲区中
* FileOutputStream：写到文件
* PipedOutputStream：管道
* FilterOutputStream：装饰器，为其他OutputStream提供有用功能
    * DataOutputStream:按照可移植方式向流中写入基本类型
    * PrintStream:产生格式化输出。内部实现用的是**BufferedWriter**，可以设置autoFlush
    * BufferedOutputStream：使用缓冲区

### OutputStreamWriter类 适配器
继承Writer类，构造器接受OutputStream对象
子类：

* FileWriter
```Java
new FileWriter("eaxmple.txt") 
//查看源码，发现其实相当于，使用的是默认encoder
new OutputStreamWriter(new FileOutputStream("eaxmple.txt"))
```

### Writer类 
子类：

* FilterWriter 
* BufferedWriter
* PrintWriter：格式化


### 典型输出使用方式
```Java
//字符文件
File file = new File("example.txt");
if (file.exists()) {
	try {
        FileOutputStream stream = new FileOutputStream(file);//OutputSteam
        OutputStreamWriter adapter = new OutputStreamWriter(stream，"GBK");//适配器
        BufferedWriter writer = new BufferedWriter(adapter);//Reader
        writer.write("test");
        writer.flush();//此处可省略
        writer.close();
    } catch (IOException e) {
    	e.printStackTrace();
    }
}
```
* 在reader.readLine()等用完后，reader.flush()是将缓冲区的内容进行实际的IO操作，如果不需要多次flush刷新的话，可以省略。因为最后reader.close()时会自动调用flush方法。
* PrintWriter的构造器除了和Writer子类一样能接受Writer对象，还接受OutputStream作为参数来构造。Java SE5之后还增加了一个构造器：
```Java
File file = new File("example.txt");
new PrintWriter(file)
//查看源码，发现其实相当于如下过程。只是为了少写点装饰过程的代码。
new PrintWriter(
    new BufferedWriter(
        new OutputStreamWriter(
            new FileOutputStream(file)))
```

### RandomAccessFile
类似C语言中的fopen()，在nio中有新的处理方式，暂不详细看这个类。

## 文件读写实用工具
装饰器使得文件读写变得麻烦，PrintWriter简化了输出的过程。而自己也可以封装一些针对Text等常用文件的操作。

虽然PrintWriter可以对数据格式化，便于阅读。但是数据的存储还是建议用DataOutputStream写入，DataInputStream恢复：
```Java
//存储数据：写入文件
File file = new File("example.txt");
if (file.exists()) {
	try {
        FileOutputStream stream = new FileOutputStream(file);
		BufferedOutputStream buffer = new BufferedOutputStream(stream);
		DataOutputStream dataOutputStream = new DataOutputStream(buffer);
		dataOutputStream.writeUTF("That was pi");
		dataOutputStream.writeDouble(3.14159);
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

```Java
//恢复数据：从文件读出
File file = new File("example.txt");
if (file.exists()) {
	try {
		FileInputStream stream = new FileInputStream(file);
		BufferedInputStream buffer = new BufferedInputStream(stream);
		DataInputStream dataInputStream = new DataInputStream(buffer);
		System.out.println(dataInputStream.readUTF());
		System.out.println(dataInputStream.readDouble());
	} catch (IOException e) {
		e.printStackTrace();
	}
}
```
## 标准IO

* System.in:public final static InputStream in
    * 需要装饰InputStream->InputStreamReader->BufferedReader
* System.out:public final static PrintStream out
    * PrintStream是OutputStream。而PrintWriter可以接受OutputStream。所以可以PrintStream->PrintWriter
* System.err:public final static PrintStream err
* 标准IO的重定向
    * System.setIn(文件输入如BufferedInputStream到标准输入)
    * System.setOut(标准输出定向到文件如PrintStream)
    * System.setErr(标准输出定向到文件如PrintStream)

## 进程控制
```Java
Process process = new ProcessBuilder(command_str.split(" ")).start();
process.getInputStream()得到一个process标准输出的InputStream
```
## 新IO
java.nio.\*包引入新的io库，并重新实现了java.io.\*
待续


## 对象序列化
Java对对象的序列化和反序列化
待续

## 压缩 

* FilterInputStream：InputStream的子类，装饰器的接口
    * CheckedInputStream：为InputStream计算CheckSum
    * InflaterInputStream：压缩类的基类
        * ZipInputStream：Zip压缩
        * GZIPInputStream：GZIP压缩
* FilterOutputStream：装饰器
    * CheckedOutputStream：为OutputStream计算CheckSum
    * DeflaterOutputStream：解压缩的基类
        * ZipOutputStream:解压缩Zip
        * GZIPOutputStream:解压缩GZIP

### GZIP
```Java
//字节流 输入
File file = new File("example.gz");
FileInputStream stream = new FileInputStream(file);
GZIPInputStream gzip = new GZIPInputStream(stream);
InputStreamReader adapter = new InputStreamReader(gzip);
BufferedReader reader = new BufferedReader(adapter);
String tmp = reader.readLine();
```
```Java
//字符流 输出
File file = new File("example.gz");
FileOutputStream stream = new FileOutputStream(file);
GZIPOutputStream gzip = new GZIPOutputStream(stream);
BufferedOutputStream writer = new BufferedOutputStream(adapter);
writer.write("this is a test");
```
### ZIP
Java的Zip库更全面
```Java
//压缩
File file = new File("example.zip");
FileOutputStream stream = new FileOutputStream(file);
CheckedOutputStream checksum = new CheckedOutputStream(stream, new Adler32());//加入校验码Adler32(),或者CRC32()等
ZipOutputStream zip = new ZipOutputStream(checksum);//Zip输出流
BufferedOutputStream buffer = new BufferedOutputStream(zip);
zip.setComment("a test");//加入评论等
ArrayList<String> filenames = new ArrayList<>();
filenames.add("example.txt");
//Zip文件中的子文件或者目录filename为"data.txt"或者"/directory"
for (String filename : filenames) {
	zip.putNextEntry(new ZipEntry(filename));
	//此时文件的buffer里指针在新的ZipEntry的起始位置，可以写入
	int byteData = -1;
	buffer.write(byteData);
	buffer.flush();
}
buffer.close();
```
```Java
//解压缩
File file = new File("example.zip");
FileInputStream stream = new FileInputStream(file);
CheckedInputStream checksum = new CheckedInputStream(stream, new Adler32());
ZipInputStream zip = new ZipInputStream(checksum);
BufferedInputStream buffer = new BufferedInputStream(zip);
ZipEntry zipEntry;
while(zipEntry = = zip.getNextEntry() != null){
//getNextEntry()会使得指针调到下一个ZipEntry的起始位置
    int byteData;
    while(byteData = buffer.read() != null) System.out.write(x);
}
System.out.print(checksum.getChecksum().getValue());
buffer.close();
```
* 对于读Zip的操作，可以用ZipFile对象读文件，再用entries()方法来向ZipEntries返回枚举
* Zip格式也被用于Java档案文件JAR(Java ARchive)

## XML
XML是一种更为通用的方法。java本身有java.xml.*类库。书中采用XOM库。
本质是XML文件的解析和编写过程，对应为程序的读写。
root - child结构
```XML
<?xml version="1.0" encoding="ISO-8859-1"?>
<people>
    <person>
        <first>Dr. Bunsen</first>
        <last>Honeydew</last>
    </person>
    <person>
        <first>Gonzo</first>
        <last>Res</last>
    </person>
</people>
```
## Preferences
Preferences API是一个键值对集合，通常用于创建以类名命名的单一节点。
如：
```Java
public class PreferencesDemo{
    public static void main(String[] args){
        Preferences prefs = Preferences.userNodeForPackage(PreferencesDemo.class);//用户
        //PreferencesDemo.class在此处作为标识节点
        //Preferences.systemNodeForPackage()最好用于通用配置
        prefs.put("Location","shenzhen");
        prefs.putInt("Companions",4);
        for(String key:prefs.keys()){
            print(key+":"+prefs.get(key,null));//put default value
        }
    }
}


/*output:
Location:shenzhen
Companions:4
*/
```
