# Android艺术开发探索——第二章：IPC机制（下）
---

>我们继续来讲IPC机制，在本篇中你将会学习到

- ContentProvider
- Socket
- Binder连接池


##一.使用ContentProvider
>ContentProvider是Android中提供的专门用来不同应用之间数据共享的方式，从这一点来看，他天生就是适合进程间通信，和Messenger一样，ContentProvider的底层实现同样也是Binder，由此可见，Binder在Android系统中是何等的重要，虽然ContentProvider的底层实现是Binder，但是他的使用过程比AIDL简单多了，这是因为系统为我们封装了，使得我们无须关心底层实现即可轻松实现IPC，ContentProvider虽然使用起来很简单，包括自己创建一个ContentProvider也不是什么难事，尽管如此，它的细节还是相当多，比如CRUD操作，防止SQL注入和权限控制等。由于章节主题限制，在本节中，笔者暂时不对ContentProvider的使用细节以及工作机制进行详细分析，而是为读者介绍采用ContentProvider进行跨进程通信的主要流程，至于使用细节和内部工作机制会在后续章节进行详细分析。

>系统预置了许多ContentProvider，比如通讯录信息、日程表信息等，要跨进程访问这些信息，只需要通过ContentResolver的query,update、insert 和 delete方法即可，在本节中，我们来实现一个自定义的ContentProvider，并演示如何在其他应用中获取ContentProvider中的数据从而实现进程间通信这一目的。首先，我们创建一个ContentProvider名字就叫BookProvider。创建一个自定义的ContentProvider很简单，只需要继承ContentProvider并且实现它的六个方法：onCreate、query、update、 insert和getType,这六个抽象方法都很好理解，onCreate代表ContentProvider的创建，一般我们要做一些初始化工作；getIype用来返回一个Uri请求的MIME类型（媒体类型，比如图片），这个媒体类型还是比较复杂的，如果我们的应用不关注这些选项，可以直接在这个方法中返回null或者/，剩下的四个方法对应于CRUD操作，即实现对数据表的增删查改功能，除了Binder的工作原理，我们知道这六个方法均运行在ContentProvider的进程中，除了onCreate由系统回调并并运行在主线程中，其他五个方法均由外界回调并且运行在Binder线程池中，这一点我们再接下来的例子中可以看到。


>ContentProvider主要以表格的形式来组织数据，并且可以包含多个表，对于每个表格来说，它们都具有行和列的层次性，行往往对应一条记录，而列对应一条记录中的一个字段，这点和数据库很类似。除了表格的形式，ContentProvider还支持文件数据，比如图片、视频等。文件数据和表格数据的结构不同，因此处理这类数据时可以在ContentProvider中返回文件的句柄给外界从而让文件来访问Contentprovider中的文件信息。Android系统所提供的MediaStore功能就是文件类型的ContentProvider，详细实现可以参考MediaStore。另外，虽然ContentProvide的底层数据看起来很像一个SQLite数据库，但是ContentProvider对底层的数据存储方式没有任何要求，我们既可以使用SQLite数据库，也可以使用普通的文件，甚至可以采用内存中的一个对象来进行数据的存储，这一点在后续的章节中会再次介绍，所以这里不再深入了。

>下面看一个最简单的示例，它演示了ContentProvider的工作工程。首先创建一个BookProvider类，它继承自ContentProvider并实现了ContentProvider的六个必须需要实现的抽象方法。在下面的代码中，我们什么都没干，尽管如此，这个BookProvider也是可以
工作的，只是它无法向外界提供有效的数据而已。

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   BookProvider
 *  创建者:   LGL
 *  创建时间:  2016/10/20 13:49
 *  描述：    ContentProvider
 */

import android.content.ContentProvider;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.support.annotation.Nullable;
import android.util.Log;

public class BookProvider extends ContentProvider{

    public static final String TAG = "BookProvider";

    @Override
    public boolean onCreate() {
        Log.i(TAG,"onCreate,current thread:" + Thread.currentThread().getName());
        return false;
    }

    @Nullable
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        Log.i(TAG,"query,current thread:" + Thread.currentThread().getName());
        return null;
    }

    @Nullable
    @Override
    public String getType(Uri uri) {
        Log.i(TAG,"getType");
        return null;
    }

    @Nullable
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        Log.i(TAG,"insert");
        return null;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        Log.i(TAG,"delete");
        return 0;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        Log.i(TAG,"update");
        return 0;
    }
}


```

>接着我们需要注册这个BookProvider，如下所示。其中android:authorities是ContenttProvider的唯一标识，通过这个属性外部应用就可以访问我们的BookProvider，因此android:authorities必须是唯一的，这里建议读者在命名的时候加上包名前缀。,为了演示进程间通讯，我们让BookProvider运行在独立的进程中并给它添加了权限，这样外界应用如果想访问BookProvider，就必须声明com.lgl.PROVIDER这个权限。ContentProvider的的权限还可以细分为读权限和写权限，分别对应androidreadPermission和
androidswritePermission 属性，如果分别声明了读权限和写权限，那么外界应用也必须依次声明相应的权限才可以进行读/写操作，否则外界应用会异常终止。关于权限这一块，请读者自行查阅相关资料，本章不进行详细介绍。

```
		<provider
            android:name=".BookProvider"
            android:authorities="com.liuguilin.contentprovidersampler.BookProvider"
            android:permission="com.lgl.PROVIDER"
            android:process=":provider"/>

```

>注册了ContentProvider之后，我们就可以在外部应用中访问他了，为了方便演示，这里我们再统一个应用中其他进程去访问这个BookProvider,和其他应用中的访问效果一样，读者可以自行试下（要声明权限）

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   ProviderActivity
 *  创建者:   LGL
 *  创建时间:  2016/10/20 13:55
 *  描述：    ContentProvider类
 */

import android.net.Uri;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;

public class ProviderActivity extends AppCompatActivity{

    @Override
    protected void onCreate( Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_provider);

        Uri uri = Uri.parse("content//com.liuguilin.contentprovidersampler.BookProvider");
        getContentResolver().query(uri,null,null,null,null);
        getContentResolver().query(uri,null,null,null,null);
        getContentResolver().query(uri,null,null,null,null);
    }
}


```
>在上面的代码中，我们通过ContentResolver对象的query方法去查询BookProvider中的数据，其中“content//com.liuguilin.contentprovidersampler.BookProvider”唯一标识了BookProvider，而这
个标识正是我们前面为BookProvider的android:authorities属性所指定的值。我们运行后看一下 log。从下面log可以看出，BookProvider中的query方法被调用了三次，并且这三次调用不在同一个线程中。可以看出，它们运行在一个Binder线程中，前面提到update、insert和delete方法同样也运行在Binder线程中。另外，onCreate运行在main线程中，也就是
UI线程，所以我们不能在onCreate中做耗时操作。

>到这里，整个ContentProvider的流程我们已经跑通了，虽然ContentProvider中没有返回任何数据。接下来，在上面的基础上，我们继续完善BookProvider，从而使其能够对外应用提供数据，继续本章提出的那个例子，现在我们要提供一个BookProvider，外部应用可以通过BookProvider来访问图书信息，为了更好地演示ContentProvider，还可以通过BookProvider访问到用户信息。为了完成上述功能，我们需要一个数据库来来管理图书和用户信息，这个数据库不难实现，代码如下，

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   DbOPenHelper
 *  创建者:   LGL
 *  创建时间:  2016/10/20 13:58
 *  描述：    数据库
 */

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class DbOPenHelper extends SQLiteOpenHelper {

    public static final String DB_NAME = "book_provider.db";
    public static final String BOOK_TABLE_NAME = "book";
    public static final String USER_TABLE_NAME = "user";

    public static final int DB_VERSION = 1;

    //图书和用户信息表
    private String CREATE_BOOK_TABLE = "CREATE TABLE ID NOT EXISTS" + BOOK_TABLE_NAME + "(_id INTEGER PRIMARY KEY," + "name TEXT)";

    private String CREATE_USER_TABLE = "CREATE TABLE IF NOT EXISTS" + USER_TABLE_NAME + "(_id INTEGER PRIMARY KEY,"+"name TEXT,";

    public DbOPenHelper(Context context) {
        super(context, DB_NAME, null, DB_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_BOOK_TABLE);
        db.execSQL(CREATE_USER_TABLE);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {

    }
}


```

>上述代码是一个最简单的数据库的实现，我们借助SQLiteOpenHelper来管理数据库的创建、升级和降级。下面我们就要通过BookProvider向外界提供上述数据库中的信息了,我们知道，ContentProvider通过Uri来区分外界要访问的的数据集合，在本例中支持
对BookProvider中的book表和user表进行访问，为了知道外界要访问的是哪个表，我需要为它们定义单独的Uri和Uri_Code，并将Uri和对应的Uru_Code相关联，我们可以用UriMatcher的addURI方法将Uri和Ur_Code关联到一起。这样，当外界请求访问BookProvider时，我们就可以根据请求的Uri来得到Ur_Code，有了Uri_Code我们就知道外界想要访问哪个表，然后就可以进行相应的数据操作了，具体代码如下

```
public class BookProvider extends ContentProvider {

    public static final String TAG = "BookProvider";

    public static final String AUTHORITY = "com.liuguilin.contentprovidersampler.BookProvider";

    public static final Uri BOOK_CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/book");

    public static final Uri USER_CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/user");

    public static final int BOOK_URI_CODE = 0;

    public static final int USER_URI_CODE = 1;

    public static final UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);
    
    static {
        sUriMatcher.addURI(AUTHORITY,"book",BOOK_URI_CODE);
        sUriMatcher.addURI(AUTHORITY,"user",USER_URI_CODE);
    }
    ....
  }

```

>从上面的代码来看，我们可以通过如下的方式来获取外界所要访问的数据源，根据Uri先取出Uri_code，关联的都是0和1，这个关联过程就是一句话

```
 sUriMatcher.addURI(AUTHORITY,"book",BOOK_URI_CODE);
 sUriMatcher.addURI(AUTHORITY,"user",USER_URI_CODE);

```

>将Uri和uri_code管理好之后，我们可以通过如下方式来获取外界需要访问的数据，根据Uri先取出uri_code，根据Uri_code再来得到表的名称，接下来我么可以响应外界的增删查改请求了

```
private String getTableName(Uri uri) {
        String tableName = null;
        switch (sUriMatcher.match(uri)) {
            case BOOK_URI_CODE:
                tableName = DbOPenHelper.BOOK_TABLE_NAME;
                break;
            case USER_URI_CODE:
                tableName = DbOPenHelper.USER_TABLE_NAME;
                break;
        }
        return tableName;
    }

```

>接着我们就可以实现增删查改的方法了，如果是qurey，首先我们要从拿到外界要访问的表名称，然后根据外界传递的信息进行数据库的查询操作了，这个过程比较简单：

```
    
    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        Log.i(TAG, "query,current thread:" + Thread.currentThread().getName());
        String table = getTableName(uri);
        if(table == null){
            throw new IllegalArgumentException("Unsupported URI:" + uri);
        }
        return mDb.query(table,projection,selection,selectionArgs,null,null,sortOrder,null);
    }

```

>另外三个方法的实现思路和查询有点类似，只有一点不同，那就是这三个方法都会引起数据源的改变，这个时候我们需要通过ContentResolver的notifyChange中的数据改变情况，可以通过注册的方法来注册观察者，对于这三个方法，这里不再详细说，看代码：

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   BookProvider
 *  创建者:   LGL
 *  创建时间:  2016/10/20 13:49
 *  描述：    ContentProvider
 */

import android.content.ContentProvider;
import android.content.ContentValues;
import android.content.Context;
import android.content.UriMatcher;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.net.Uri;
import android.support.annotation.Nullable;
import android.util.Log;

public class BookProvider extends ContentProvider {

    public static final String TAG = "BookProvider";

    public static final String AUTHORITY = "com.liuguilin.contentprovidersampler.BookProvider";

    public static final Uri BOOK_CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/book");

    public static final Uri USER_CONTENT_URI = Uri.parse("content://" + AUTHORITY + "/user");

    public static final int BOOK_URI_CODE = 0;

    public static final int USER_URI_CODE = 1;

    public static final UriMatcher sUriMatcher = new UriMatcher(UriMatcher.NO_MATCH);

    static {
        sUriMatcher.addURI(AUTHORITY, "book", BOOK_URI_CODE);
        sUriMatcher.addURI(AUTHORITY, "user", USER_URI_CODE);
    }

    private Context mContext;
    private SQLiteDatabase mDb;

    @Override
    public boolean onCreate() {
        Log.i(TAG, "onCreate,current thread:" + Thread.currentThread().getName());
        mContext = getContext();
        initProviderDate();
        return true;
    }

    private void initProviderDate() {
        mDb = new DbOPenHelper(mContext).getWritableDatabase();
        mDb.execSQL("delete from " + DbOPenHelper.BOOK_TABLE_NAME);
        mDb.execSQL("delete from " + DbOPenHelper.USER_TABLE_NAME);

        mDb.execSQL("insert into book values(3,'Android');");
        mDb.execSQL("insert into book values(4,'IOS');");
        mDb.execSQL("insert into book values(5,'Html5');");
        mDb.execSQL("insert into book values(1,'jake',1);");
        mDb.execSQL("insert into book values(2,'Jasmine',0);");
    }


    @Override
    public Cursor query(Uri uri, String[] projection, String selection, String[] selectionArgs, String sortOrder) {
        Log.i(TAG, "query,current thread:" + Thread.currentThread().getName());
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI:" + uri);
        }
        return mDb.query(table, projection, selection, selectionArgs, null, null, sortOrder, null);
    }

    @Nullable
    @Override
    public String getType(Uri uri) {
        Log.i(TAG, "getType");
        return null;
    }

    @Nullable
    @Override
    public Uri insert(Uri uri, ContentValues values) {
        Log.i(TAG, "insert");
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI:" + uri);
        }
        mDb.insert(table, null, values);
        mContext.getContentResolver().notifyChange(uri, null);
        return uri;
    }

    @Override
    public int delete(Uri uri, String selection, String[] selectionArgs) {
        Log.i(TAG, "delete");
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI:" + uri);
        }
        int count = mDb.delete(table, selection, selectionArgs);
        if (count > 0) {
            getContext().getContentResolver().notifyChange(uri, null);
        }
        return count;
    }

    @Override
    public int update(Uri uri, ContentValues values, String selection, String[] selectionArgs) {
        Log.i(TAG, "update");
        String table = getTableName(uri);
        if (table == null) {
            throw new IllegalArgumentException("Unsupported URI:" + uri);
        }
        int row = mDb.update(table, values, selection, selectionArgs);
        if (row > 0) {
            getContext().getContentResolver().notifyChange(USER_CONTENT_URI, null);
        }
        return row;
    }

    private String getTableName(Uri uri) {
        String tableName = null;
        switch (sUriMatcher.match(uri)) {
            case BOOK_URI_CODE:
                tableName = DbOPenHelper.BOOK_TABLE_NAME;
                break;
            case USER_URI_CODE:
                tableName = DbOPenHelper.USER_TABLE_NAME;
                break;
        }
        return tableName;
    }
}

```

>需要注意的是，增删查改四大方法是存在多线程并发访问的，因此方法内部要做好线程同步的工作，在本例中，由于采取了sqlite并且只有一个SQLiteDatabase内部对数据库的操作式同步处理的，但是如果多个SQLiteDatabase对象来操作数据库就无法保证线程同步了，因为SQLiteDatabase对象之间无法进程线程同步，如果ContentProvider的底层数据集是一块内存的话，比如List，在这种情况下同List的遍历，插入，删除操作就需要进行线程同步了，否则会引发错误，这点尤其需要注意的，到这里BookProvider已经完成了，接着我们来外部访问他，看看他能否继续工作

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   ProviderActivity
 *  创建者:   LGL
 *  创建时间:  2016/10/20 13:55
 *  描述：    ContentProvider类
 */

import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.support.v7.app.AppCompatActivity;

public class ProviderActivity extends AppCompatActivity{

    public static final String TAG = "ProviderActivity";
    
    @Override
    protected void onCreate( Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_provider);

        Uri bookUri = Uri.parse("content//com.liuguilin.contentprovidersampler.BookProvider/book");

        ContentValues values = new ContentValues();
        values.put("_id",6);
        values.put("name","程序设计的艺术");
        getContentResolver().insert(bookUri,values);
        Cursor bookCursor = getContentResolver().query(bookUri,new String[]{"_id","name"},null,null,null);
        while (bookCursor.moveToNext()){
            Book book = new Book();
            book.id = bookCursor.getInt(0);
            book.name = bookCursor.getString(1);
        }
        bookCursor.close();
        
        Uri userUri = Uri.parse("content//com.liuguilin.contentprovidersampler.BookProvider/user");
        Cursor userCursor = getContentResolver().query(userUri,new String[]{"_id","name","sex"},null,null,null);
        while (userCursor.moveToNext()){
            User user = new User();
            user.id= userCursor.getInt(0);
            user.name = userCursor.getString(1);
            user.isMale = userCursor.getInt(2) == 1;
        }
        userCursor.close();
        
    }
}


```

>默认情况下，BookProvider的数据库中有三本书和两个用户，在上面的代码中，我们首先添加一本书：“程序设计的艺术”。接着查询所有的图书，这个时候应该查询出四本书，，因为我们刚刚添加了一本。然后查询所有的用户，这个时候应该查询出两个用户。是不是这样呢?我们运行一下程序，从log可以看到，我们的确查询到了4本书和2个用户，这说明BookProvider已经能够正确地处理外部的请求了，读者可以自行验证一下update和delete操作，这里就不再验证了。同时，由于ProviderActivity和BookProvider运行在两个不同的进程中，因此，这也构成了进程间的通信。ContentProvider除了支持对数据源的增删改查这四个操作，还支持自定义调用，这个过程是通ContentResolver的Call方法和ContentProvider的Call方法来完成的。关于使用ContentProvider来进行IPC就介绍到这里，ContentProvider本身还有一些细节这里并没有介绍，读者可以自行了解，本章侧重的是各种进程间通信的方法以及它们的区别，因此针对某种特定的方法可能不会介绍得面面俱到。另外，ContentProvider在后续章节还会有进一步的讲解，主要包括细节问题和工作原理，读者可以阅读后面的相应章节

##二.使用Socket
>在本节，我们通过Socket来实现进程通信，Socket也叫做套接字，是网络通信中的概念，他分为流式套接字和用户数据报套接字两种，分别是应用于网络的传输控制层中的Tcp和UDP协议，TCP面向的连接协议，提供稳定的双向通讯功能，TCP连接的建立需要经过“三次握手”才能完成，为了提供稳定的数据传输功能，其本身提供了超时重传机制，因此具有很高的稳定性：而UDP是无连接的，提供不稳定的单向通信功能，当然UDP也可以实现双向通信功能。在性能上，UDP具有更好的效率，其缺点是不保证数据一定能够正确传输，尤其是在网络拥塞的情况下。关于TCP和UDP的介绍就这么多，更详细的资料请查看相关网络资料。接下来我们演示一个跨进程的聊天程序，两个进程可以通过Socket来实现信息的传输，Socket本身可以支持传输任意字节流，这里为了简单起见，仅仅传输文本信息，很显然，这是一种IPC方式。

>使用Socket来进行通信，有两点需要注意，首先需要声明权限：


```
 <uses-permission android:name="android.permission.INTERNET"/>
 <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
```

>其次要注意不能在主线程中访问网络，因为这会导致我们的程序无法在Android4.0及其以上的设备中运行，会抛出如下异常：android.os NetworkOnMainThreadException。而且进行网络操作很可能是耗时的，如果放在主线程中会影响程序的响应效率，从这方面来说，也不应该在主线程中访问网络。下面就开始设计我们的聊天室程序了，比较简单，首先在远程Service建立一个TCP服务，然后在主界面中连接TCP服务，连接上了以后，就可给服务端发消息。对于我们发送的每一条文本消息，服务端都会随机地回应我们一句话为了更好地展示Socket的工作机制，在服务端我们做了处理，使其能够和多个客户端同时连接建立连接并响应。

>先看一下服务端的设计，当Service启动时，会在线程中建立TCP服务，这里监听的是8688端口，然后就可以等待客户端的连接请求。当有客户端连接时，就会生成一个新的Socket，通过每次新创建的Socket就可以分别和不同的客户端通信了。服务端每收到一次客户端的消息就会随机回复一句话给客户端。当客户端断开连接时，服务端这边也会相应的关闭对应Socket并结束通话线程，这点是如何做到的呢?方法有很多，这里是通过判断服务端输入流的返回值来确定的，当客户端断开连接后，服务端这边的输入流会返回null,这个时候我们就知道客户端退出了。服务端的代码如下所示。

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   TCPServerService
 *  创建者:   LGL
 *  创建时间:  2016/10/22 15:16
 *  描述：    服务端
 */

import android.app.Service;
import android.content.Intent;
import android.os.IBinder;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.Random;

public class TCPServerService extends Service {

    private boolean mIsServiceDestoeyed = false;
    private String[] mDefinedMessages = {"你好呀", "你叫神马？", "今天的天气", "汽车站怎么走？"};

    @Override
    public void onCreate() {
        new Thread(new TcpServer()).start();
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }

    @Override
    public void onDestroy() {
        mIsServiceDestoeyed = true;
        super.onDestroy();
    }
    
    private class TcpServer implements Runnable{

        @Override
        public void run() {
            ServerSocket serverSocket = null;
            try {
                //监听本地8868端口号
                serverSocket = new ServerSocket(8688);
            } catch (IOException e) {
                e.printStackTrace();
                return;
            }
            
            while (!mIsServiceDestoeyed){
                try {
                    //接收客户端的请求
                    final Socket client = serverSocket.accept();
                    new Thread(){
                        @Override
                        public void run() {
                            try {
                                responseClient(client);
                            } catch (IOException e) {
                                e.printStackTrace();
                            }
                        }
                    }.start();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    private void responseClient(Socket client) throws IOException{
        //用于接收客户端的信息
        BufferedReader in = new BufferedReader(new InputStreamReader(client.getInputStream()));
        //用于给客户端发送消息
        PrintWriter out = new PrintWriter(new BufferedWriter(new OutputStreamWriter(client.getOutputStream())),true);
        out.print("欢迎来到聊天室");
        while (!mIsServiceDestoeyed){
            String str = in.readLine();
            if(str == null){
                break;
            }
            int i = new Random().nextInt(mDefinedMessages.length);
            String msg = mDefinedMessages[i];
            out.print(msg);
        }
        in.close();
        out.close();
        client.close();
    }
}

```

>接下来看一下客户端，客户端Activity启动时，会在onCreate中开启一个线程去连接服务端的socket，至于为什么要用线程我们前面已经说了，为了确定能够连接成功，这里采用了超时重连的机制，每次连接失败后都会重新连接，当然，为了降低重试机制的开销，我们加入了休眠机制，每次重试的事件间隔为1000毫秒

```
	 try {
            //接收服务端的消息
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            while (!TCPClientActivity.this.isFinishing()) {
                String msg = br.readLine();
                if (msg != null) {
                    String time = formatDateTime(System.currentTimeMillis());
                    String showMsg = "server " + time + "：" + msg + "\n";
                    handler.obtainMessage(MESSAGE_RECEIVE_NEW_MSG, showMsg).sendToTarget();
                }
            }


```

>当然，你的activity在退出的时候，就要关闭socket了


```
  @Override
    protected void onDestroy() {
        if (mClientSocket != null) {
            try {
                mClientSocket.shutdownInput();
                mClientSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        super.onDestroy();
    }
```

>接收发送消息的整个过程，这个就很简单了，看完整代码：


```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   TCPClientActivity
 *  创建者:   LGL
 *  创建时间:  2016/10/22 15:31
 *  描述：    客户端
 */

import android.content.Intent;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.support.v7.app.AppCompatActivity;
import android.text.TextUtils;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;
import java.net.Socket;
import java.text.SimpleDateFormat;
import java.util.Date;

public class TCPClientActivity extends AppCompatActivity implements View.OnClickListener {

    public static final int MESSAGE_RECEIVE_NEW_MSG = 1;
    public static final int MESSAGE_SOCKET_CONNECTED = 2;

    private Button mSendButton;
    private TextView mMessageTextView;
    private EditText mMessageEditText;

    private PrintWriter mPrintWriter;
    private Socket mClientSocket;

    private Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MESSAGE_RECEIVE_NEW_MSG:
                    mMessageTextView.setText(mMessageTextView.getText() + (String) msg.obj);
                    break;
                case MESSAGE_SOCKET_CONNECTED:
                    mSendButton.setEnabled(true);
                    break;
            }
        }
    };

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_tcp_client);
        initView();
    }

    private void initView() {
        mMessageTextView = (TextView) findViewById(R.id.msg_container);
        mSendButton = (Button) findViewById(R.id.send);
        mSendButton.setOnClickListener(this);
        mMessageEditText = (EditText) findViewById(R.id.msg);

        Intent intent = new Intent(this, TCPServerService.class);
        startService(intent);

        new Thread() {
            @Override
            public void run() {
                connectTCPServer();
            }
        }.start();
    }


    @Override
    protected void onDestroy() {
        if (mClientSocket != null) {
            try {
                mClientSocket.shutdownInput();
                mClientSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        super.onDestroy();
    }

    @Override
    public void onClick(View v) {
        if (v == mSendButton) {
            String msg = mMessageEditText.getText().toString();
            if (!TextUtils.isEmpty(msg)) {
                mPrintWriter.println(msg);
                mMessageEditText.setText("");
                String time = formatDateTime(System.currentTimeMillis());
                String showesMsg = "self" + time + ":" + msg + "\n";
                mMessageTextView.setText(mMessageTextView.getText() + showesMsg);
            }
        }
    }

    private String formatDateTime(long l) {
        return new SimpleDateFormat("(HH:mm:ss)").format(new Date(l));
    }

    private void connectTCPServer() {
        Socket socket = null;
        while (socket == null) {
            try {
                socket = new Socket("localhost", 8688);
                mClientSocket = socket;
                mPrintWriter = new PrintWriter(new BufferedWriter(new OutputStreamWriter(socket.getOutputStream())), true);
                handler.sendEmptyMessage(MESSAGE_SOCKET_CONNECTED);
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        try {
            //接收服务端的消息
            BufferedReader br = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            while (!TCPClientActivity.this.isFinishing()) {
                String msg = br.readLine();
                if (msg != null) {
                    String time = formatDateTime(System.currentTimeMillis());
                    String showMsg = "server " + time + "：" + msg + "\n";
                    handler.obtainMessage(MESSAGE_RECEIVE_NEW_MSG, showMsg).sendToTarget();
                }
            }

            mPrintWriter.close();
            br.close();
            socket.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}


```

>上述代码就是通过Socket来进行进程间的通信实例，除了采用套接字，还可以用UDP套接字，OK

##三.Binder连接池
>上面我们介绍了不同的IPC方式，我们知道，不同的IPC方式有不同的特点和适用场景,当然这个问题会在26节进行介绍，在本节中要再次介绍一下AIdL，原因是AIDL是一种最常用的进程间通信方式，是日常开发中涉及进程间通信时的首选，所以我们需要额外强调一下.

>如何使用AIDL在上面的一节中已经进行了介绍，这里在回顾一下大致流程：首先创建一个Service和一个AIDL接口，接着创建一个类继承自AIDL接口中的Stub类并实现Stub中的抽象方法，在Service的onBind方法中返回这个类的对象，然后客户端就可以绑
定服务端Service，建立连接后就可以访问远程服务端的方法了。上述过程就是典型的AIDL的使用流程。这本来也没什么问题，但是现在考虑一种情况；公司的项目越来越庞大了，现在有10个不同的业务模块都需要使用AIDL来进行进程间通信，那我们该怎么处理呢？也许你说：“就按照AIDL的实现方式一个个来吧”，这是可以的，如果用这种方法，首先我们需要创建10个Service，这好像有点多啊！如果有100个地方需要用到AIDL呢，先创建100个Servlce?到这里，读者应该明白问题所在了，随着AIDL数量的增加，我们不能无限制地增Service,Service是四大组件之一，本生是一种系统资源。而且太多的Serice会使得我们的应用看起来很重量级，因为正在运行的Service可以在应用详情页看到，当我们的应用详情显示有10个个服务正在运行时，这看起来并不是什么好事。针对上述问题，我们需要减少Service的数量，将所有的AIDL放在同个Service中去管理。

>在这种模式下，整个工作机制是这样的；每个业务模块创建自己的AIDL接口并实现此接口，这个时候不同业务模块之间是不能有耦合的，所有实现细节我们要单独开来，然后向服务端提供自己的唯一标识和其对应的Binder对象；对于服务端来说，只需要一个Service就可以了，服务端提供一个queryBinder接口，这个接口能够根据业务模块的特征来返回相应的Binder对象给它们，不同的业务模块拿到所需的Binder对象后就可以进行远程方法调用了。由此可见，Binder连接池的主要作用就是将每个业务模块的Binder请求统一转发到远程Service中去执行，从而避免了重复创建Service的过程，它的工作原理如图所示。

![这里写图片描述](http://img.blog.csdn.net/20161022191707960)


>通过上面的理论介绍，也许还有点不好理解，下面对Binder连接池的代码实现做一说明。首先，为了说明问题，我们提供了两个AIDL接口（ISecurityCenter和ICompute）来模拟上面提到的多个业务模块都要使用AIDL的情况，其中ISecurityCenter接口提供解密功能，声明如下：

```
interface ISecurityCenter {
    String encrypt(String content);
    String decrypt(String password);
}
```

>而ICompute提供了计算加法的功能：

```
interface ICompute {

    int add(int a,int b);
}


```

>虽然说上面的两个接口的功能都比较简单，但是用于分析Binder池的工作原理还是足够的，读者可以写出更加复杂的例子，接下来我们来看一下部分

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   SecurityCenterImpl
 *  创建者:   LGL
 *  创建时间:  2016/10/22 17:50
 *  描述：    TODO
 */

import android.os.RemoteException;

public class SecurityCenterImpl extends ISecurityCenter.Stub{

    private static  final  char SECRET_CODE = '^';

    @Override
    public String encrypt(String content) throws RemoteException {
        char [] chars = content.toCharArray();
        for (int i = 0; i < chars.length; i++) {
            chars[i] ^= SECRET_CODE;
        }
        return new String(chars);
    }

    @Override
    public String decrypt(String password) throws RemoteException {
        return encrypt(password);
    }
}



```


```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   ComputeImpl
 *  创建者:   LGL
 *  创建时间:  2016/10/22 17:52
 *  描述：    TODO
 */

import android.os.RemoteException;

public class ComputeImpl extends ICompute.Stub {

    @Override
    public int add(int a, int b) throws RemoteException {
        return a + b;
    }
}


```

>现在的业务模块的AIDL接口定义和实现都已经完成了，注意的是这里并没有为每个模块的AIDL创建单独的Service,接下来就是服务端和Binder连接池的工作了

```

interface IBinderPool {
    
    IBinder queryBinder(int binderCode);
}


```

>接着，为Binder连接池创建远程Service并实现IBnderPool,下面是queryBinder的具体实现，可以看到请求转达的实现方法，当Binder连接池连接上远程服务时，会根据不同的模块的标识binderCode返回不同的Binder对象，通过这个对象就可以操作全部发生在远程服务端上：

```
 public static class BinderPoolImpl extends IBinderPool.Stub{
        
        public BinderPoolImpl(){
            super();
        }

        @Override
        public IBinder queryBinder(int binderCode) throws RemoteException {
            IBinder binder = null;
            switch (binderCode){
                case BINDER_SECURUITY_CENTER:
                    binder = new SecurityCenterImpl();
                    break;
                case BINDER_COMPUTE:
                    binder = new ComputeImpl();
                    break;
            }
            return binder;
        }
    }

```

>远程service的实现比较简单，如下：

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   BinderPoolService
 *  创建者:   LGL
 *  创建时间:  2016/10/22 18:01
 *  描述：    Binder
 */

import android.app.Service;
import android.content.Intent;
import android.os.Binder;
import android.os.IBinder;

public class BinderPoolService extends Service{

    private  static final String  TAG = "BinderPoolService";

    private Binder mBinderPool = new BinderPool.BinderPoolImpl();

    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return mBinderPool;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}

```

>下面还剩下Binder连接池的具体实现了，在他的内部首先他要去绑定远程服务，绑定成功后，客户端就课堂通过他的queryBinder方法来获取对应的Binder，拿到所需的Binder之后，不同业务模块就可以各自操作了


```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   BinderPool
 *  创建者:   LGL
 *  创建时间:  2016/10/22 18:04
 *  描述：    TODO
 */

import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.os.IBinder;
import android.os.RemoteException;

import java.util.concurrent.CountDownLatch;

public class BinderPool {

    private static final String TAG = "BinderPool";
    public static final int BINDER_NONE = -1;
    public static final int BINDER_COMPUTE = 0;
    public static final int BINDER_SECURUITY_CENTER = 1;

    private Context mContext;
    private IBinderPool mIBinderPool;
    private static volatile BinderPool sInstance;
    private CountDownLatch mConnectBinderPoolCopuntDownLacth;

    private BinderPool(Context context) {
        mContext = context.getApplicationContext();
        connectBinderPoolService();
    }

    public static BinderPool getInstance(Context context) {
        if (sInstance == null) {
            synchronized (BinderPool.class) {
                if (sInstance == null) {
                    sInstance = new BinderPool(context);
                }
            }
        }
        return sInstance;
    }

    private synchronized void connectBinderPoolService() {
        mConnectBinderPoolCopuntDownLacth = new CountDownLatch(1);
        Intent service = new Intent(mContext, BinderPoolService.class);
        mContext.bindService(service, mBinderPoolConnection, Context.BIND_AUTO_CREATE);
        try {
            mConnectBinderPoolCopuntDownLacth.await();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }

    public IBinder queryBinder(int binderCode) {
        IBinder binder = null;
        try {
            if (mIBinderPool != null) {
                binder = mIBinderPool.queryBinder(binderCode);
            }
        } catch (RemoteException e) {
            e.printStackTrace();
        }
        return binder;
    }

    private ServiceConnection mBinderPoolConnection  = new ServiceConnection() {

        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mIBinderPool = IBinderPool.Stub.asInterface(service);
            try {
                mIBinderPool.asBinder().linkToDeath(mBinderPoolDeathRecipient,0);
            } catch (RemoteException e) {
                e.printStackTrace();
            }
            mConnectBinderPoolCopuntDownLacth.countDown();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

   private IBinder.DeathRecipient mBinderPoolDeathRecipient  = new IBinder.DeathRecipient() {
       @Override
       public void binderDied() {
           mIBinderPool.asBinder().unlinkToDeath(mBinderPoolDeathRecipient,0);
           mIBinderPool = null;
           connectBinderPoolService();
       }
   };

    public static class BinderPoolImpl extends IBinderPool.Stub{

        public BinderPoolImpl(){
            super();
        }

        @Override
        public IBinder queryBinder(int binderCode) throws RemoteException {
            IBinder binder = null;
            switch (binderCode){
                case BINDER_SECURUITY_CENTER:
                    binder = new SecurityCenterImpl();
                    break;
                case BINDER_COMPUTE:
                    binder = new ComputeImpl();
                    break;
            }
            return binder;
        }
    }
}

```

>Binder连接池就具体的分析完了，他的好处显而易见，针对上面的例子，我们只需要创建一个Service就可以完成多个AIDL的工作，我们现在可以来验证一下他的功能，新创建一个Activity，在线程中执行如下的操作

```
package com.liuguilin.contentprovidersampler;

/*
 *  项目名：  ContentProviderSampler 
 *  包名：    com.liuguilin.contentprovidersampler
 *  文件名:   BinderActivity
 *  创建者:   LGL
 *  创建时间:  2016/10/22 18:26
 *  描述：    TODO
 */

import android.os.Bundle;
import android.os.IBinder;
import android.os.RemoteException;
import android.support.v7.app.AppCompatActivity;

public class BinderActivity extends AppCompatActivity{

    private ISecurityCenter mSecurityCenter;
    private ICompute mCompute;

    @Override
    protected void onCreate( Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_binder);

        new Thread(new Runnable() {
            @Override
            public void run() {
                dowork();
            }
        }).start();
    }

    private void dowork() {
        BinderPool binderPool = BinderPool.getInstance(BinderActivity.this);
        IBinder securityBinder = binderPool.queryBinder(BinderPool.BINDER_SECURUITY_CENTER);
        mSecurityCenter = SecurityCenterImpl.asInterface(securityBinder);
        String msg = "Android";
        try {
            String password = mSecurityCenter.encrypt(msg);
            System.out.print(mSecurityCenter.decrypt(password));
        } catch (RemoteException e) {
            e.printStackTrace();
        }

        IBinder computeBinder =  binderPool.queryBinder(BinderPool.BINDER_COMPUTE);
        mCompute = ComputeImpl.asInterface(computeBinder);
        try {
            System.out.print(mCompute.add(3,5));
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}

```

>这里需要额外说明一下，为什么要在线程中去执行呢？这是因为在Binder连接池的实现中，我们通过CountDownLatch将bindService这一异步操作转换成了同步操作，这就意味着他有可能是耗时的，然后就是Binder方法的调用过程也可能是耗时的，因此不建议放在主线程中执行。注意到BindePool是一个单例实现，因此在同一个进程中只会初始化一次，所以如果我们提前初始化BinderPool，那么可以优化程序的体验，比如我们可以放在Application中提前对BinderPool进行初始化，虽然这不能保证当我们调用BinderPool时它一定是初始化，好的，但是在大多数情况下，这种初始化工作（绑定远程服务）的时间开销（如果Binderpool没有提前初始化完成的话)是可以接受的。另外，BinderPool中有断线重连的机制，当远程服务意外终止时，BinderPool会重新建立连接，这个时候如果业务模块中的Binder调用出了异常，也需要手动去重新获取最新的Binder对象，这个是需要注意的。

>有了BinderPool可以大大方便日常的开发工作，比如如果有一个新的业务模块需要添加新的AIDL，那么在他实现自己的AIDL接口后，只需要修改BinderPoolImpl中的queryBinder方法，给自己添加新的binderCode并返回对应的Binder对象就可以，不需要做其他的修改，野不需要创建新的Service，由此可见，BinderPool能够极大的提高对AIDL的开发效率，并且可以避免大量的Service创建，因此比较建议使用


##四.选用是个自己的IPC方式

>在上面的一节中，我们介绍了各种各样的IPC方式，那么到底它们有什么不同呢?我到底该使用哪一种呢?本节就为读者解答这些问题，具体内容如表所示。通过表可以明确地看出不同IPC方式的优缺点和适用场景，那么在实际的开发中，只要我们选适的IPC方式就可以轻松完成多进程的开发场景。

![这里写图片描述](http://img.blog.csdn.net/20161022191720795)

