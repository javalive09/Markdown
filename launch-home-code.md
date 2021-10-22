# **应用中启动Home的正确方式**

对于启动Home的intent，在AMS中有判断，如下：
```
private boolean isHomeIntent(Intent intent) {
        return ACTION_MAIN.equals(intent.getAction())
                && intent.hasCategory(CATEGORY_HOME)
                && intent.getCategories().size() == 1 
                && intent.getData() == null 
                && intent.getType() == null;

}
```

也就是说，intent的action必须是Intent.ACTION_MAIN，categories必须是CATEGORY_HOME，不能setData()，不能setType()。
否则虽然也可以启动Home，但是Home是会放到普通的stack中，并不会放到正常Launcher应该在的stack中，这时候Launcher是作为一个普通activity存在的，在某些情况下，会存在问题。