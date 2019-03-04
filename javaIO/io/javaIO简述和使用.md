## 一、流的概念
  流（stream）的概念源于Unix中管道的概念。在Unix中管道是一条不间断的字节流，用于程序和进程之间的通信，或读写外围设备，外部文件。一个流必须有源端和目的端，它们可以是计算机内存，也可以是磁盘文件，甚至可以是Internet上的某个URL。
  
  流的方向是重要的，根据流的方向。流可以分为两类：输入流和输出流。用户可以从输入流读取信息，但不能写它。相反，对于输出流，只能往里写，而不能读它。实际上，流的源端和目的端可简单地看成是字节的生产者和消费者，对输入流，可不必关心它的源端是什么，只要简单地从流中读数据，而对输出流，也可不知道它的目的端，只是简单地往流中写数据。 
  
## 二、流的分类
  java IO 包中的stream类根据操作的对象是字节还是字符，分为字节流和字符流；
  根据数据流向不同分为：输入流（`InputStream`）和输出流（`OutputStream`）。
  
  
### 1. java字符流和字符流：
  
  `InputStream`是所有字节输入流的祖先，`OutputStream`是所有字节输出流的祖先；
  
  `Reader`是所有读取字符串的祖先，`Writer`是所有输出字符串的祖先
  
  字符流的由来：因为数据编码的不同，而有了对字符进行高效操作的流对象。本质是基于字符流读取时，去查了指定的码表。
  
  
### 2. 字节流和字符流的区别：
  
  (1)读写单位不同：字节流以字节(8bit)为单位，字符流以字符为单位，根据码表映射字符，一次可能读取多个字节。
  
  (2)处理对象不同：字节流能处理所有类型的数据（如图片、avi等），而字符流只能处理字符类型数据。
  
  (3)字符流在操作的时候本身是不会用到缓冲区的，是文件本身直接操作的；而字符流在操作的时候回用到缓冲区，通过缓冲区来操作文件。
  
### 3. java字节流 

  从输入字节流的继承图可以看出：

  `InputStream` 是所有的输入字节流的父类，它是一个抽象类。

  `ByteArrayInputStream`、`StringBufferInputStream`、`FileInputStream` 是三种基本的介质流，它们分别从Byte 数组、StringBuffer、和本地文件中读取数据。`PipedInputStream` 是从与其它线程共用的管道中读取数据，与Piped 相关的知识后续单独介绍。

  `ObjectInputStream` 和所有`FilterInputStream`的子类都是装饰流（装饰器模式的主角）。意思是`FileInputStream`类可以通过一个String路径名创建一个对象，`FileInputStream(String name)`。而`DataInputStream`必须装饰一个类才能返回一个对象，`DataInputStream(InputStream in)`。如下图示:

  ![image](https://github.com/fengmuhai/Hello_Github/blob/master/images/java.io/inputstream-relation.png)
  
##### 为了说明输入输出流对文件的操作，一些demo将会结合输出流给出示例
  
#####[示例1] FileInputStream读取文件内容
###### 与`FileInputStream`从文件中读数据一样，`ByteArrayStream`、`StringBufferString`是从byte[]、StringBuffer对象中读取数据
```java
import java.io.*;
public class ReadFile {
	
	public static void main(String[] args) throws Exception {
		String path = "E:"+File.separator+"test.txt";
		System.out.println(readFile(path));
	}
	
	public static String readFile(String file) throws Exception {
		InputStream input = null;
		byte[] buf = new byte[128];
		int len = 0;
		StringBuilder sb = new StringBuilder("");
		input = new FileInputStream(file);		//这里的参数file可以是文件业务可以是路径
		while((len=input.read(buf)) != -1) {
			sb.append(readByteBuffer(buf,0,len));
			//sb.append(new String(buf,0,len));
		}
		input.close();
		return sb.toString();
	}
		
	public static char[] readByteBuffer(byte[] buf, int start, int end) {
		if(buf==null || buf.length<=0 || start<0 || end<=0 ||start>=end) {
			return new char[1];
		}
		char[] ch = new char[end-start];
		for(int i=start;i<end;i++) {
			ch[i-start] = (char) buf[i];
		}
		return ch;
	}

}
```

上面的`readByteBuffer`是自己实现的将byte转换成char的方法，可以直接使用new String()来实现。经过测试，自己实现的方法比较快，不过还可以
通过把ch定义成静态的变量、优化参数判断等来提升执行速度。

其实，读取文件可以直接通过FileReader对象来实现，也可以通过`InputStreamReader`对象来操作，它们读取的是字符，更容易转化为字符串：参考4.java字符流 [案例 ]和[案例 ]


##### [示例2] ObjectOutputStream写对象文件 ObjectInputStream读取文件内容
该流允许读取或写入用户自定义的类，但是要实现这种功能，被读取和写入的类必须实现`Serializable`接口，（这里结合两者使用）实例代码如下：
```java
public class ObjetStream {  

   public static void main(String[] args) {  
	   String path = "E:"+File.separator+"test.txt";
	   objectStreamTest(path);
   }  
   
   public static void objectStreamTest(String file) {
	   ObjectOutputStream objectwriter=null;  
	   ObjectInputStream objectreader=null;  
	   try {  
		   objectwriter=new ObjectOutputStream(new FileOutputStream(file));  
		   objectwriter.writeObject(new Student("gg", 22));  
		   objectwriter.writeObject(new Student("tt", 18));  
		   objectwriter.writeObject(new Student("rr", 17));  
		   objectreader=new ObjectInputStream(new FileInputStream(file));  
		   for (int i = 0; i < 3; i++) {  
			   System.out.println(objectreader.readObject());  
		   }  
	   } catch (Exception e) {   
		   e.printStackTrace();  
	   }finally{  
		   try {  
			   objectreader.close();  
			   objectwriter.close();  
		   } catch (IOException e) {  
			   e.printStackTrace();  
		   }  

	   }  
   }
   
}  
class Student implements Serializable{  
	private static final long serialVersionUID = 1L;
	private String name;  
	private int age;  
    
	public Student(String name, int age) {  
		super();  
		this.name = name;  
		this.age = age;  
   }  
   
   @Override  
   public String toString() {  
	   return "Student [name=" + name + ", age=" + age + "]";  
   }  
   
}  
```

##### [示例3] DataInputStream/DataOutputStream 读取/写入文件
有时没有必要存储整个对象的信息，而只是要存储一个对象的成员数据，成员数据的类型假设都是Java的基本数据类型，这样的需求不必使用到与Object输入、输出相关的流对象，可以使用`DataInputStream`、`DataOutputStream`来写入或读出数据。
下面是一个例子：（`DataInputStream`的好处在于在从文件读出数据时，不用费心地自行判断读入字符串时或读入int类型时何时将停止，使用对应的readUTF()和readInt()方法就可以正确地读入完整的类型数据。）
```java
public class DataStream {  

	public static void main(String[]args)  {  
		String path = "E:"+File.separator+"test.txt";
		dataStreamTest(path);
	}
	
	public static void dataStreamTest(String file) {
		Member[] members = {new Member("Justin",90), new Member("momor",95), new Member("Bush",88)};  
		try {  
			DataOutputStream dataOutputStream = new DataOutputStream(new FileOutputStream(file));  
			for(Member member:members) {  
				dataOutputStream.writeUTF(member.getName());  //写入UTF字符串 
				dataOutputStream.writeInt(member.getAge());  //写入int数据  
			}  
			dataOutputStream.flush();  //所有数据至目的地  
			dataOutputStream.close();  //关闭流  

			DataInputStream dataInputStream = new DataInputStream(new FileInputStream(file));  
			for(int i=0;i<members.length;i++) {  //读出数据并还原为对象  
				String name =dataInputStream.readUTF();  //读出UTF字符串
				int score =dataInputStream.readInt();  //读出int数据  
				members[i] = new Member(name,score);  
			}  
			dataInputStream.close();  //关闭流  
			for(Member member : members) { //显示还原后的数据  
				System.out.printf("%s\t%d%n",member.getName(),member.getAge());  
			}  
		}  
		catch(IOException e) {  
			e.printStackTrace();  
		}  
		
	}
		
}  

class Member {  
	private String name;  
	private int age;  
	public Member() {  
	}  
	public Member(String name, int age) {  
		this.name = name;  
		this.age = age;  
	}  
	public void setName(String name){  
		this.name = name;  
	}  
	public void setAge(int age) {  
		this.age = age;  
	}  
	public String getName() {  
		return name;  
	}  
	public int getAge() {  
		return age;  
	}  
}  
```

##### [示例5] PushBackInputStream操作流
`PushbackInputStream`类继承了`FilterInputStream`类是iputStream类的修饰者。提供可以将数据插入到输入流前端的能力(当然也可以做其他操作)。简而言之`PushbackInputStream`类的作用就是能够在读取缓冲区的时候提前知道下一个字节是什么，其实质是读取到下一个字符后回退的做法，这之间可以进行很多操作，这有点向你把读取缓冲区的过程当成一个数组的遍历，遍历到某个字符的时候可以进行的操作，当然，如果要插入，能够插入的最大字节数是与推回缓冲区的大小相关的，插入字符不能大于缓冲区，示例如下：
```java
public class PushBackInputStream {

	public static void main(String[] args) throws IOException {  
	    String str = "hello,rollenholt";  
	    PushbackInputStream push = null;  
	    ByteArrayInputStream bat = null;  
	    bat = new ByteArrayInputStream(str.getBytes());  
	    push = new PushbackInputStream(bat); 	// 创建回退流对象，将拆解的字节数组流传入  
	    int temp = 0;  
	    while ((temp = push.read()) != -1) {	 // push.read()逐字节读取存放在temp中，如果读取完成返回-1  
	       if (temp == ',') {
	          push.unread(temp);	 //回到temp的位置  
	          temp = push.read(); 	//接着读取字节  
	          System.out.print("(回退" + (char) temp + ") ");	 // 输出回退的字符  
	       } else {  
	          System.out.print((char) temp); // 否则输出字符  
	       }  
	    }  
	}  
}
```

##### [示例6] SequenceInputStream操作流对象
有些情况下，当我们需要从多个输入流中向程序读入数据。此时，可以使用合并流，将多个输入流合并成一个`SequenceInputStream`流对象。`SequenceInputStream`会将与之相连接的流集组合成一个输入流并从第一个输入流开始读取，直到到达文件末尾，接着从第二个输入流读取，依次类推，直到到达包含的最后一个输入流的文件末尾为止。 合并流的作用是将多个源合并合一个源。其可接收枚举类所封闭的多个字节流对象。
```java
public class SequenceInputStream() {
	//待完成
}
```


### (2) OutputStream
  IO 中输出字节流的继承图可见下图，可以看出：

  OutputStream 是所有的输出字节流的父类，它是一个抽象类。

  `ByteArrayOutputStream`、`FileOutputStream`是两种基本的介质流，它们分别向Byte 数组、和本地文件中写入数据。`PipedOutputStream` 是向与其它线程共用的管道中写入数据。

  `ObjectOutputStream` 和所有`FilterOutputStream`的子类都是装饰流。具体例子跟InputStream是对应的。
  ![image](https://github.com/fengmuhai/Hello_Github/blob/master/images/java.io/outputstream.png)
  
##### [示例2.1]



### 4. java字符流

#### [案例2] FileReader读取文件内容
```java
public static String xmlFileToString(String path) {
	FileReader reader = null;
	StringBuilder sb = new StringBuilder();

	char[] buf = new char[1024];
	try {
		reader = new FileReader(path);
		int ch = 0;
		while((ch=reader.read(buf))!=-1) {
			//将数组转化为字符串打印,后面参数的意思是
			//如果字符数组未满，转化成字符串打印后尾部也许会出现其他字符
			//因此，读取的字符有多少个，就转化多少为字符串
			sb.append(String.valueOf(buf,0,ch));
		}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		try {
			reader.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	return sb.toString();
}
```

#### [案例3] FileInputStreamReader读取文件
```java
public static String xmlFileToString2(String path) {
	FileInputStream in = null;
	InputStreamReader reader = null;
	StringBuilder sb = new StringBuilder();

	char[] buf = new char[1024];
	try {
		in = new FileInputStream((new File(path)));
		reader = new InputStreamReader(in, "UTF-8");
		int ch = 0;
		while((ch=reader.read(buf))!=-1) {
			sb.append(String.valueOf(buf,0,ch));
		}
	} catch (IOException e) {
		e.printStackTrace();
	} finally {
		try {
			in.close();
			reader.close();
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	return sb.toString();
}
```

  
  
  
  
  
  
  
  
  
  
  
