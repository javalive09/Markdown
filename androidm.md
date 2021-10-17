# AndroidM

## 数据库

### 数据库设计范式

#### 第一范式

字段不可再分

#### 第二范式

一个表只表示一个事物

#### 第三范式

每一列都与主键有直接关系，不存在传递依赖。

### sqlite数据库

#### 使用

```text
1.extend SQLiteOpenHelper
2. new xx()
3. .getWirtableDatabase();
```

#### 常用操作

**建表**

CREATE TABLE 表名 \(列名1 类型，列名2 类型\)

```text
create table if not exists person(persionid integer primary key autoincrement, name varchar(20));
```

```text
db.execSQL("CREATE TABLE person (personid integer primary key autoincrement, name varchar(20))");
```

**修改表**

ALTER TABLE 表名 ADD 列名 类型 NULL

```text
alter table person add phone varchar(20);
```

```text
db.execSQL("ALTER TABLE person ADD phone VARCHAR(12) NULL ");
```

**insert**

insert into 表名 （列名1，列名2 ）values （值1，值2）；

```text
insert into person(name,phone) values('peter','123');
```

```text
db.execSQL("insert into person (name,phone) values (?,?)", new Object[] { "peter", "123"});
```

```text
ContentValues values = new ContentValues();  
values.put("name", name);  
values.put("age", age);  
// 如果 contentvalues为空  
db.insert("person", null, values); // 组拼sql语句完成的添加的操作
```

**delete**

delete from 表名 where 条件

```text
delete from person where name = "peter";
```

```text
db.execSQL("delete from person where name =?", new Object[] { "peter"});
```

```text
db.delete("person", "name=?", new String[] { name });
```

**update**

```text
update person set name='p',phone='456' where name='peter';
```

```text
db.execSQL("update person set name=? , phone=? where name=?", new Object[] { newname, newphone, name });
```

```text
ContentValues values = new ContentValues();  
values.put("name", newname);  
values.put(“phone”, newphone);  
db.update("person", values, "name=?", new String[] { name });
```

**查询信息**

select 列名 from 表名 where 条件

```text
select * from person where name="peter";
```

```text
db.rawQuery("select * from person where name=?", new String[] { name });
```

```text
Cursor cursor = db.query("person", null, "name=?", new String[] { name }, null, null, null);
```

**查询个数**

select count\(column\) from 表名

```text
select count(*) from person;
```

```text
db.rawQuery("select count(*) from person", null);
```

```text
if(cursor.moveToNext()) {  
    int index = cursor.getColumnIndex("columnName");  
    int count = cursor.getInt(index);  
}
```

**清空表**

```text
db.execSQL("delete from "+ TABLE_NAME);
```

```text
db.delete(TABLE_NAME, null, null);
```

需要返回删除数量可以用

```text
db.delete(TABLE_NAME, "1", null);
```

#### 设计数据库时需要考虑的问题

**数据库升级**

**多线程访问**

**加密**

**速度优化**

#### getWritableDatabase\(\)和getReadableDataBase\(\)的区别

一般情况下return相同对象。在磁盘满的情况下。getReadableDataBase\(\) return 只读对象。getWritableDataBase\(\)抛出异常。

#### 升级的逻辑

1. 数据库文件version    ==  openhelper 设置db的Version 。 开启数据库。
2. 数据库文件version    !=    openhelper 设置db的Version。

    a. == 0  oncreate\(\)

    b. &gt;     onDowngrade\(\)

    c. &lt;     onUpgrade\(\) 

### CursorAdapter刷新机制

今天看downloadmanager源码发现 cursorAdapter 会在数据变化的情况下没有调用notifydatasetxx\(\) 而是自动刷新。和其他adapeter刷新方式不一样。 看了一下源码，发现cursoradapter内部有自动的刷新代码。如下：

```text
void init(Context context, Cursor c, int flags) {  
    if ((flags & FLAG_AUTO_REQUERY) == FLAG_AUTO_REQUERY) {  
        flags |= FLAG_REGISTER_CONTENT_OBSERVER;  
        mAutoRequery = true;  
    } else {  
        mAutoRequery = false;  
    }  
    boolean cursorPresent = c != null;  
    mCursor = c;  
    mDataValid = cursorPresent;  
    mContext = context;  
    mRowIDColumn = cursorPresent ? c.getColumnIndexOrThrow("_id") : -1;  
    if ((flags & FLAG_REGISTER_CONTENT_OBSERVER) == FLAG_REGISTER_CONTENT_OBSERVER) {  
        mChangeObserver = new ChangeObserver();  
        mDataSetObserver = new MyDataSetObserver();  
    } else {  
        mChangeObserver = null;  
        mDataSetObserver = null;  
    }  

    if (cursorPresent) {  
        if (mChangeObserver != null) c.registerContentObserver(mChangeObserver);  
        if (mDataSetObserver != null) c.registerDataSetObserver(mDataSetObserver);  
    }  
}
```

observer代码中会进行数据更新，如下：

```text
private class MyDataSetObserver extends DataSetObserver {  
    @Override  
    public void onChanged() {  
        mDataValid = true;  
        notifyDataSetChanged();  
    }  

    @Override  
    public void onInvalidated() {  
        mDataValid = false;  
        notifyDataSetInvalidated();  
    }  
}
```

## SharedPreference

### 使用方法1

Activity中getPreferences\(int mode\)。 xml文件名：\(activity类全名去掉包名\).xml

```text
public SharedPreferences getPreferences(int mode) {  
    return getSharedPreferences(getLocalClassName(), mode);  
}
```

```text
public String getLocalClassName() {  
    final String pkg = getPackageName();  
    final String cls = mComponent.getClassName();  
    int packageLen = pkg.length();  
    if (!cls.startsWith(pkg) || cls.length() <= packageLen  
            || cls.charAt(packageLen) != '.') {  
        return cls;  
    }  
    return cls.substring(packageLen+1);  
}
```

### 使用方法2

context中getSharedPreferences\(String name, int mode\)。xml文件名：name.xml

```text
@Override  
public SharedPreferences getSharedPreferences(String name, int mode) {  
    return mBase.getSharedPreferences(name, mode);  
}
```

### 使用方法3

PreferenceManager中getDefaultSharedPreferencesName\(Context context\) xml文件名：包名\_preferences.xml

```text
public static SharedPreferences getDefaultSharedPreferences(Context context) {  
    return context.getSharedPreferences(getDefaultSharedPreferencesName(context),  
            getDefaultSharedPreferencesMode());  
}  

private static String getDefaultSharedPreferencesName(Context context) {  
    return context.getPackageName() + "_preferences";  
}
```

