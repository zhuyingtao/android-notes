### Android 数据库

Android系统内置了SQLite数据库，并且提供了一整套的API用于对数据库进行增删改查操作。

每个应用的数据库存储目录： `/data/data/<package_name>/databases`



#### 数据库加密

1. 使用 SQLCipher 加密

SQLCipher是一个在SQLite基础之上进行扩展的开源数据库，它主要是在SQLite的基础之上增加了数据加密功能，可以大大提高程序的安全性。

```java
import android.content.Context;  
import net.sqlcipher.database.SQLiteDatabase;  
import net.sqlcipher.database.SQLiteDatabase.CursorFactory;  
import net.sqlcipher.database.SQLiteOpenHelper;  
  
public class MyDatabaseHelper extends SQLiteOpenHelper {  
      
    public static final String CREATE_TABLE = "create table Book(name text, pages integer)";  
  
    public MyDatabaseHelper(Context context, String name, CursorFactory factory, int version) {  
        super(context, name, factory, version);  
    }  
  
    @Override  
    public void onCreate(SQLiteDatabase db) {  
        db.execSQL(CREATE_TABLE);  
    }  
  
    @Override  
    public void onUpgrade(SQLiteDatabase db, int arg1, int arg2) {  
  
    }  
}  
```

除了引入的包不一样了，其它的用法和传统的SQLiteOpenHelper都是完全相同的。

在使用时，有如下代码：

```java
SQLiteDatabase.loadLibs(this);  
MyDatabaseHelper dbHelper = new MyDatabaseHelper(this, "demo.db", null, 1);  
db = dbHelper.getWritableDatabase("secret_key");  
```

首先调用了SQLiteDatabase的loadLibs()静态方法将SQLCipher所依赖的so库加载进来，然后创建MyDatabaseHelper的实例，并调用getWritableDatabase()方法去获取SQLiteDatabase对象。这里在调用getWritableDatabase()方法的时候传入了一个字符串参数，它就是SQLCipher所依赖的key，在对数据库进行加解密的时候SQLCipher都将使用这里指定的key。