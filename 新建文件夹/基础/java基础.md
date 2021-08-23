**java基础**

String的方法



1.返回指定子字符串首次出现在该字符串中的索引。

```
public int indexOf(String str)
```

​	使用所有值.indexOf(包含值)，返回第一次出现包含值的地方。类型为int。



2.截取字符串

```
public String[] split(String regex) split(String str)
```

​	按照str将字符串截取成数组	使用	String[] sss=s.split(" ");



3.字符串转换

​	字符串转int

```
public static int parseInt(String s)
```

​	使用	int a = Integer.parseInt(aa)；

​	

​	int转字符串

```
public static String valueOf(Object obj);
```

​	使用	String s2 = String.valueOf(i);