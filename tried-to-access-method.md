# **混淆引起的java.lang.IllegalAccessError: tried to access method问题追踪**
# 一，背景
xxmanager 修改并混淆后会随机出现java.lang.IllegalAccessError: tried to access method问题。
# 二，定位原因
## 1，发现xx-frames.jar和xx-server.jar存在两个全路径名相同的类 com.xx.utils.a.class
## 2，java类加载机制
ClassLoader.java
```
protected Class<?> loadClass(String name, boolean resolve)
    throws ClassNotFoundException
{
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            try {
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
 
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                c = findClass(name);
            }
        }
        return c;
}
```
## 3，关键代码
```
// First, check if the class has already been loaded
Class<?> c = findLoadedClass(name);
```
```
/**
 * Returns the class with the given <a href="#name">binary name</a> if this
 * loader has been recorded by the Java virtual machine as an initiating
 * loader of a class with that <a href="#name">binary name</a>.  Otherwise
 * <tt>null</tt> is returned.
 *
 * @param  name
 *         The <a href="#name">binary name</a> of the class
 *
 * @return  The <tt>Class</tt> object, or <tt>null</tt> if the class has
 *          not been loaded
 *
 * @since  1.1
 */
protected final Class<?> findLoadedClass(String name) {
    ClassLoader loader;
    if (this == BootClassLoader.getInstance())
        loader = null;
    else
        loader = this;
    return VMClassLoader.findLoadedClass(loader, name);
}
```
```
@FastNative
native static Class findLoadedClass(ClassLoader cl, String name);
```
findLoadedClass 为native实现，从findLoadedClass的注释中可以看出如果加载过这个类，则直接返回并不会重新加载。
所以问题点出现在这。由于虚拟机已经加载过xx-frames.jar的 com.xx.utils.a.class 所以在xx-server.jar中调用com.xx.utils.a.class时会直接调用xx-frames.jar中加载过的class而这个类与wt-server.jar不在同一个包下于是产生了访问权限异常。