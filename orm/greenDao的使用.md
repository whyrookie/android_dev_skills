## GreenDao基本使用-入门

GreenDao应该是一个很受欢迎的orm框架，很早以前使用过，现在已经是3.0了，这里做个记录。
先看下官网上说的优点:
性能最好（可能是android平台最快的orm框架）
简单易用
GreenDao体积小
数据库加密:支持SQLCipher
强大的社区属性(开源，star多)

项目地址:https://github.com/greenrobot/greenDAO

gradle配置:

project.gradle:

buildscript {
    repositories {
        jcenter()
        mavenCentral() // add repository
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath 'org.greenrobot:greendao-gradle-plugin:3.2.2' // add plugin
    }
}

app.gradle:

apply plugin: 'com.android.application'
apply plugin: 'org.greenrobot.greendao' // apply plugin

dependencies {
    compile 'org.greenrobot:greendao:3.2.2' // add library
}

greendao {
    schemaVersion 2//greenDAO架构版本，相当于数据库版本
}
使用：
Note Entity:代表了一个在数据库中保存的类(表)，这个类的属性相当于数据库表中列的字段。

```java
@Entity(indexes = {
        @Index(value = "text, date DESC", unique = true)//索引
})
public class Note {

    @Id
    private Long id;

    @NotNull//text非空
    private String text;
    private String comment;
    private Date date;

    //NoteTypeConverter可以将我们定义的枚举类型和数据库的type互相装换.
    @Convert(converter = NoteTypeConverter.class, columnType = String.class)
    private NoteType type;//是个枚举类型

    @Generated(hash = 1272611929)//自动生成
    public Note() {
    }

    public Note(Long id) {
        this.id = id;
    }

    @Generated(hash = 59778150)//自动生成
    public Note(Long id, @NotNull String text, String comment, Date date, NoteType type) {
        this.id = id;
        this.text = text;
        this.comment = comment;
        this.date = date;
        this.type = type;
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getText() {
        return text;
    }

    public void setText(@NotNull String text) {
        this.text = text;
    }

    public String getComment() {
        return comment;
    }

    public void setComment(String comment) {
        this.comment = comment;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }

    public NoteType getType() {
        return type;
    }

    public void setType(NoteType type) {
        this.type = type;
    }
}
```
NoteType:
```java
public enum NoteType {
    TEXT, LIST, PICTURE
}
```

NoteTypeConverter:
```java
public class NoteTypeConverter implements PropertyConverter<NoteType, String> {

    @Override
    public NoteType convertToEntityProperty(String databaseValue) {
        return NoteType.valueOf(databaseValue);
    }

    @Override
    public String convertToDatabaseValue(NoteType entityProperty) {

        return entityProperty.name();
    }
}
```

当我们调用Build > Make project后，greenDao会自动帮我们生成greenDao类，比如这里生成的是NoteDao.java,我们根据这个Dao类来执行数据库表的各项操作.

为了操作数据库首先我们得初始化数据库，在自定义的Application类中初始化，供整个app使用.

```java
public class App extends Application {

    public static final boolean ENCRYPTED = true;//是否加密

    private DaoSession daoSession;

    @Override
    public void onCreate() {
        super.onCreate();

        //创建一个数据库，DevOpenHelper实际上最终实现了SQLiteOpenHelper，
        DaoMaster.DevOpenHelper helper = new DaoMaster.DevOpenHelper(this, "notes-db");
        Database db = ENCRYPTED ? helper.getEncryptedWritableDb("super-secret") : helper.getWritableDb();
        daoSession = new DaoMaster(db).newSession();

    }

    public DaoSession getDaoSession() {
        return daoSession;
    }


    public static App sInstance = new App();

    public static App getInstance() {
        return sInstance;
    }
}

```
Activity中获取NoteDao对象：
```java
    DaoSession daoSession = ((App)getApplication()).getDaoSession();

    noteDao  = daoSession.getNoteDao();
```


几个核心类:
DaoMaster:greenDAO的入口类，持有SQLiteDatabase对象，并且管理Dao类(不是对象)，可以创建或者删除表，它的两个内部类OpenHelper和DevOpenHelper都实现了SQLiteOpenHelper。
DaoSession:管理特殊模式下全部可用的Dao对象，我们可用通过get方法获取相应的Dao对象(代码在下面)，DaoSession提供了插入，加载，更新，刷新和删除等针对实体(Entity)的操作。
Daos:Data access objects (DAOs),我们通过这个类操作数据，相比DaoSession,它提供了更多的持久化方法，比如count, loadAll, and insertInTx。
Entities：持久化数据对象，每个对象相当于数据库表中的每条记录.

核心类初始化流程:
```java
// do this once, for example in your Application class
helper = new DaoMaster.DevOpenHelper(this, "notes-db", null);
db = helper.getWritableDatabase();//获取到读写数据库
daoMaster = new DaoMaster(db);
daoSession = daoMaster.newSession();
// do this in your activities/fragments to get hold of a DAO
noteDao = daoSession.getNoteDao();//获取的noteDao供我们操作数据
```

常规操作:

插入：
```java
private void addNote() {

        Note note = new Note();
        note.setText(noteText);
        note.setComment(comment);
        note.setDate(new Date());
        note.setType(NoteType.TEXT);
        noteDao.insert(note);

        Log.d("DaoExample", "Inserted new note, ID" + note.getId());
    }

```

删除：

```java
noteDao.deleteByKey(id);
```

更新:
```java
note.setText("This note has changed.");
noteDao.update(note);
```

查找:
```java
List<User> joes = noteDao.queryBuilder().list();
```
上面的操作都很简洁明了，查找操作内容其实比较较多，因为包括各种条件查找和多线程查找，下一篇再说(具体代码可看github上greenDao团队提供的demo)。
