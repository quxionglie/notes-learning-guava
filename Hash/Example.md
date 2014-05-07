Hash样例代码
==========

对字符串或文件md5
```
public static String md5(String str) {
        return Hashing.md5().hashBytes(str.getBytes()).toString();
    }

    public static String md5(File file) throws IOException {
        return Files.hash(file, Hashing.md5()).toString();
    }
```
