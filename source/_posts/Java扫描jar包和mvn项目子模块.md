---
title: Java扫描jar包和mvn项目子模块
date: 2020-03-08 22:06:14
tags: [java,maven,scanner]
categories: java
---

> 最近刚入职，领导分配了一个小任务：将自研的mq系统的配置全部打印出来。听起来好像挺简单，但是一看代码发现好多配置项不在一个module里，而且有的配置项置只暴露出来api，如果在不修改代码的情况下，是无法获取到所有配置项的。想了一下午，想到使用`spi`或者`asm`来解决，`spi`一样有无法获取`private`,`package-protected`或者其他模块的配置项的问题。`asm`想着应该可以解决，但我是个`asm`半吊子水货，用的不好。所以想来想去只想到扫描`mvn`项目来解决这个问题。好在已经有人遇到过这种问题并已经解决了([地址](https://blog.csdn.net/a729913162/article/details/81698109))，先说声谢谢啦。


###  定义Scanner接口
```java
public interface Scanner {
    
    String CLASS_SUFFIX = ".class";

    Set<Class<?>> search(String pkgName, Predicate<Class<?>> predicate);

    default Set<Class<?>> search(String pkgName){
        return search(pkgName,null);
    }
}
```

### 定义FileScanner
```java
public class FileScanner implements Scanner{

    private String defaultClassPath = FileScanner.class.getResource("/").getPath();

    public String getDefaultClassPath(){
        return defaultClassPath;
    }

    public void setDefaultClassPath(String defaultClassPath) {
        this.defaultClassPath = defaultClassPath;
    }

    public FileScanner(){}

    public FileScanner(String defaultClassPath){
        this.defaultClassPath=defaultClassPath;
    }

    @Override
    public Set<Class<?>> search(String pkgName, Predicate<Class<?>> predicate) {
        String classPath = defaultClassPath;
        String basePkgPath = pkgName.replace(".",File.separator);
        String searchPath = classPath+basePkgPath;
        return new ClassSearcher().doPath(new File(searchPath),pkgName,predicate,true);
    }

    private static class ClassSearcher {

        private Set<Class<?>> classPaths = new HashSet<>(0);

        private Set<Class<?>> doPath(File file,String pkgName,Predicate<Class<?>> predicate,boolean flag){
            if(file.isDirectory()){
                File[] files=file.listFiles();
                if(!flag){
                    pkgName = pkgName+"."+file.getName();
                }
                if(files!=null) {
                    for (File f : files) {
                        doPath(f, pkgName, predicate, false);
                    }
                }
            }else{
                if(file.getName().endsWith(CLASS_SUFFIX)){
                    try {
                        Class<?> clazz = Class.forName(pkgName+"."+file.getName().substring(0,file.getName().lastIndexOf(".")));
                        if(predicate==null||predicate.test(clazz)){
                            classPaths.add(clazz);
                        }
                    } catch (ClassNotFoundException e) {
                        e.printStackTrace();
                    }
                }
            }
            return classPaths;
        }
    }
}
```

### 定义JarScanner
```java
public class JarScanner implements Scanner {

    @Override
    public Set<Class<?>> search(String pkgName, Predicate<Class<?>> predicate) {
        Set<Class<?>> clazzSet = new HashSet<>(0);
        try {
            Enumeration<URL> urlEnumeration = Thread.currentThread().getContextClassLoader().getResources(pkgName.replace(".","/"));
            while (urlEnumeration.hasMoreElements()){
                URL url = urlEnumeration.nextElement();
                String protocol = url.getProtocol();
                if("jar".equalsIgnoreCase(protocol)){
                    JarURLConnection connection = (JarURLConnection) url.openConnection();
                    if(connection!=null){
                        JarFile jarFile = connection.getJarFile();
                        if(jarFile!=null){
                            Enumeration<JarEntry> jarEntryEnumeration = jarFile.entries();
                            while (jarEntryEnumeration.hasMoreElements()){
                                JarEntry entry = jarEntryEnumeration.nextElement();
                                String entryName = entry.getName();
                                if(entryName.contains(CLASS_SUFFIX)&&entryName.replaceAll("/",".").startsWith(pkgName)){
                                    String clazzName = entryName.substring(0,entryName.lastIndexOf(".")).replace("/",".");
                                    Class<?> clazz = Class.forName(clazzName);
                                    if(predicate==null||predicate.test(clazz)){
                                        clazzSet.add(clazz);
                                    }
                                }
                            }
                        }
                    }
                }
                else if("file".equalsIgnoreCase(protocol)){
                    FileScanner fileScanner = new FileScanner();
                    fileScanner.setDefaultClassPath(url.getPath().replace(pkgName.replace(".","/"),""));
                    clazzSet.addAll(fileScanner.search(pkgName,predicate));
                }
            }
        } catch (ClassNotFoundException | IOException e) {
            e.printStackTrace();
        }
        return clazzSet;
    }
}
```

### 定义两种Scanner的委托
```java
public class ScannerExecutor implements Scanner{

    private volatile static ScannerExecutor INSTANCE;

    @Override
    public Set<Class<?>> search(String pkgName, Predicate<Class<?>> predicate) {
        Scanner fileScanner = new FileScanner();
        Set<Class<?>> fileSearch = fileScanner.search(pkgName,predicate);
        Scanner jarScanner = new JarScanner();
        Set<Class<?>> jarSearch = jarScanner.search(pkgName,predicate);
        fileSearch.addAll(jarSearch);
        return fileSearch;
    }

    private ScannerExecutor(){}

    public static ScannerExecutor getInstance(){
        if(INSTANCE==null){
            synchronized (ScannerExecutor.class){
                if(INSTANCE==null){
                    INSTANCE=new ScannerExecutor();
                }
            }
        }
        return INSTANCE;
    }
}
```

### 定义封装的工具类
```java
public class ClassScanner {

    public static Set<Class<?>> search(String pkgName){
        return search(pkgName);
    }


    public static Set<Class<?>> search(String pkgName, Predicate<Class<?>> predicate){
        return ScannerExecutor.getInstance().search(pkgName,predicate);
    }
}
```