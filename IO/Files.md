Files类常用方法
=============

http://docs.guava-libraries.googlecode.com/git-history/release/javadoc/com/google/common/io/Files.html

#文件移动
###static void	copy(File from, File to)
Copies all the bytes from one file to another.

###static void	move(File from, File to)
Moves a file from one path to another.

###static void	touch(File file)
Creates an empty file or updates the last updated timestamp on the same as the unix command of the same name.

###static void	createParentDirs(File file)
Creates any necessary but nonexistent parent directories of the specified file.

###static File	createTempDir()
Atomically creates a new directory somewhere beneath the system's temporary directory (as defined by the java.io.tmpdir system property), and returns its name.
static boolean	equal(File file1, File file2)
Returns true if the files contains the same bytes.

```java
File file = Files.createTempDir();
System.out.println(file.getAbsoluteFile());
//out /var/folders/9l/scf5txbx49746ds5wf8_mgxw0000gp/T/1399364716588-0
//$ ls -ld /var/folders/9l/scf5txbx49746ds5wf8_mgxw0000gp/T/1399364775049-0
//drwxr-xr-x  2 xionglie  staff  68 May  6 16:26 /var/folders/9l/scf5txbx49746ds5wf8_mgxw0000gp/T/1399364775049-0
```

#文件转为Source/Sink
###static ByteSink	asByteSink(File file, FileWriteMode... modes)

Returns a new ByteSink for writing bytes to the given file.

###static ByteSource	asByteSource(File file)

Returns a new ByteSource for reading bytes from the given file.

###static CharSink	asCharSink(File file, Charset charset, FileWriteMode... modes)

Returns a new CharSink for writing character data to the given file using the given character set.

###static CharSource	asCharSource(File file, Charset charset)

Returns a new CharSource for reading character data from the given file using the given character set.

#获取文件名或文件扩展名
###static String	getFileExtension(String fullName)
Returns the file extension for the given file name, or the empty string if the file has no extension.

###static String	getNameWithoutExtension(String file)
Returns the file name without its file extension or path.

```java
String fullName = "/tmp/wad/sdw1234.jpg";
System.out.println(Files.getFileExtension(fullName));
System.out.println(Files.getNameWithoutExtension(fullName));
// ---output---
//jpg
//sdw1234
```

#文件hash判断
###static HashCode	hash(File file, HashFunction hashFunction)
Computes the hash code of the file using hashFunction.

```
HashCode md5 = Files.hash(file, Hashing.md5());
String hexStr = md5.toString();
```

#文件读取
###static <T> T	readBytes(File file, ByteProcessor<T> processor)
Process the bytes of a file.

###static String	readFirstLine(File file, Charset charset)
Reads the first line from a file.

###static List<String>	readLines(File file, Charset charset)
Reads all of the lines from a file.
```java
File file = new File("test.txt");
List<String> lines = null;
try {
    lines = Files.readLines(file, Charsets.UTF_8);
} catch (IOException e) {
    e.printStackTrace();
}
```
###static <T> T	readLines(File file, Charset charset, LineProcessor<T> callback)
Streams lines from a File, stopping when our callback returns false, or we have read all of the lines.

###static byte[]	toByteArray(File file)
Reads all bytes from a file into a byte array.

###static String	toString(File file, Charset charset)
Reads all characters from a file into a String, using the given character set.

