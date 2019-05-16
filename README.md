# 期中设计：NOTEPAD笔记本应用
本实验将基于NotePad应用做功能拓展

## 实验要求和目的
-基本要求
1.NoteList中显示条目增加时间戳显示
2.添加笔记查询功能（根据标题查询）
+附加功能
1.更改记事本背景
2.文本字体大小颜色修改
3.增加笔记编辑背景色
4.导出笔记至本地
###  实验步骤
#### 1.运用原本NotePad的MODIFIED_TIME进行时间戳显示
数据库中已经存在了COLUMN_NAME_CREATE_DATE、COLUMN_NAME_MODIFICATION_DATE

```
public void onCreate(SQLiteDatabase db) {
    db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
            + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
            + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
            + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"//创建时间
            + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER,"//修改时间
            + NotePad.Notes.COLUMN_NAME_TEXT_COLOR + " INTEGER,"//文本字体颜色
            + NotePad.Notes.COLUMN_NAME_TEXT_SIZE + " INTEGER,"//文字字体大小
            + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"//文本背景色
            + ");");
}
```
在NotePadProvider中将createtime和modifiedtime转换成相应所需格式：

```
// Gets the current system time in milliseconds
Long now = Long.valueOf(System.currentTimeMillis());

Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
String dateTime = format.format(date);

// If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateTime);
}

// If the values map doesn't contain the modification date, sets the value to the current
// time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
}
```
且在NoteEdirtor中的updateNote方法中对修改时间modifiedtime存入数据库前进行格式转换：

```
private final void updateNote(String text, String title) {

    // Sets up a map to contain values to be updated in the provider.
    ContentValues values = new ContentValues();
    Long now = Long.valueOf(System.currentTimeMillis());

    Date date = new Date(now);
    SimpleDateFormat format = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    format.setTimeZone(TimeZone.getTimeZone("GMT+08:00"));
    String dateTime = format.format(date);
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateTime);
```
在notelist_item.xml中增加TestView显示时间戳：

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <TextView
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"/>
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textColor="@color/black"
        android:textSize="20sp"
        />
</LinearLayout>
```
增加时间戳后结果如图：

![](https://github.com/c815852517/NotePad_plus/blob/master/app/时间戳显示.png)

#### 2.添加笔记查询功能（按标题查询）
首先在list_options_menu.xml中添加搜索：

```
?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
<!--  This is our one standard application action (creating a new note). -->
<item android:id="@+id/menu_add"
    android:icon="@drawable/ic_menu_compose"
    android:title="@string/menu_add"
    android:alphabeticShortcut='a'
    android:showAsAction="always" />
<!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
      so that the user can paste in the data.. -->
<item android:id="@+id/menu_paste"
    android:icon="@drawable/ic_menu_compose"
    android:title="@string/menu_paste"
    android:alphabeticShortcut='p' />
<item
    android:id="@+id/menu_search"
    android:icon="@android:drawable/ic_menu_search"
    android:title="搜索"
    android:showAsAction="always"
    />
</menu>
```
之后在onOptionsItemSelected的switch (item.getItemId())中添加对应menu_search的case：

```
 case R.id.menu_search:
      startActivity(new Intent(Intent.ACTION_SEARCH,getIntent().getData()));
      return true;
default:
    return super.onOptionsItemSelected(item);
```
添加Activity NoteSearch：

```
package com.example.android.notepad;

import android.app.Activity;
import android.database.Cursor;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;

public class NoteSearch extends Activity implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[]{
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示
    };
    private String[] mStrs = {"11", "22", "33", "44"};
    private SearchView mSearchView;
    private ListView lListView;
    private SimpleCursorAdapter adapter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        mSearchView = (SearchView) findViewById(R.id.search_view);
        mSearchView.setIconifiedByDefault(false);
        mSearchView.setSubmitButtonEnabled(true);
        mSearchView.setOnQueryTextListener(this);
        //mSearchView.setBackgroundColor(getResources().getColor());
        lListView = (ListView) findViewById(R.id.list);
        lListView.setAdapter(new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1, mStrs));
        lListView.setTextFilterEnabled(true);
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String s) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";//查询条件
        String[] selectionArgs = { "%"+s+"%" };//查询条件参数，配合selection参数使用,%通配多个字符

        //查询数据库中的内容,当我们使用 SQLiteDatabase.query()方法时，就会得到Cursor对象， Cursor所指向的就是每一条数据。
        //managedQuery(Uri, String[], String, String[], String)等同于Context.getContentResolver().query()
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.用于ContentProvider查询的URI，从这个URI获取数据
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date.用于标识uri中有哪些columns需要包含在返回的Cursor对象中
                selection,                        // 作为查询的过滤参数，也就是过滤出符合selection的数据，类似于SQL的Where语句之后的条件选择
                selectionArgs,                    // 查询条件参数，配合selection参数使用
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.查询结果的排序方式，按照某个columns来排序，例：String sortOrder = NotePad.Notes.COLUMN_NAME_TITLE
        );

        //一个简单的适配器，将游标中的数据映射到布局文件中的TextView控件或者ImageView控件中
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text2 };
        adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,          // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        lListView.setAdapter(adapter);
        return true;
    }
}

```
添加相应布局note_search.xml：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:queryHint="请输入搜索内容..."
        android:iconifiedByDefault="true">
    </SearchView>
    <ListView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/list"></ListView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
添加搜索功能后结果截图：

![](https://github.com/c815852517/NotePad_plus/blob/master/app/搜索1.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/搜索2.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/搜索3.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/搜索4.png)

#### 3.更改记事本背景
把NoteLis背景色从t黑色换成白色，在AndroidManifest.xml中NotesList的Activity中添加：

```
<activity
    android:name=".NotesList"
    android:label="@string/title_notes_list"
    android:theme="@android:style/Theme.Holo.Light">
```
效果如图：

![](https://github.com/c815852517/NotePad_plus/blob/master/app/搜索1.png)

#### 4.文字字体大小及颜色修改
先在editor_options_menu.xml添加相应的更改字体颜色大小：

```
<item
    android:id="@+id/font_size"
    android:title="字体大小">
    <!--子菜单-->
    <menu>
        <!--定义一组单选菜单项-->
        <group>
            <!--定义多个菜单项-->
            <item
                android:id="@+id/font_10"
                android:title="10号字体"
                />

            <item
                android:id="@+id/font_16"
                android:title="16号字体" />
            <item
                android:id="@+id/font_20"
                android:title="20号字体" />
        </group>
    </menu>
</item>

    <item
        android:title="字体颜色"
        android:id="@+id/font_color"
        >
        <menu>
            <!--定义一组普通菜单项-->
            <group>
                <!--定义两个菜单项-->
                <item
                    android:id="@+id/red_font"
                    android:title="红色" />
                <item
                    android:title="黑色"
                    android:id="@+id/black_font"/>
                <item
                    android:title="蓝色"
                    android:id="@+id/blue_font"/>
            </group>
        </menu>
    </item>
```
然后在NoteEditor的onOptionsItemSelected(MenuItem item)中添加相应的case来相应事件：

```
case R.id.font_10:
    mText.setTextSize(20);
    Toast toast =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast.show();
    break;

case R.id.font_16:
    mText.setTextSize(32);
    Toast toast2 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast2.show();
    break;
case R.id.font_20:
    mText.setTextSize(40);
    Toast toast3 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast3.show();
    break;
case R.id.red_font:
    mText.setTextColor(Color.RED);
    Toast toast4 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast4.show();
    mCursor.moveToFirst();
    color=mCursor.getInt(NotePad.Notes.RED_COLOR);
    ContentValues values = new ContentValues();
    values.put(NotePad.Notes.COLUMN_NAME_TEXT_COLOR, color);
    getContentResolver().update(mUri, values, null, null);
    mCursor.close();
    break;
case R.id.black_font:
    mText.setTextColor(Color.BLACK);
    Toast toast5 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast5.show();
    break;
case R.id.blue_font:
    mText.setTextColor(Color.BLUE);
    Toast toast6 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
    toast6.show();
    break;
```
实现效果如图：

![](https://github.com/c815852517/NotePad_plus/blob/master/app/字体1.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/字体2.png)

#### 5.增加笔记本编辑背景色
数据库中添加背景色字段：

```
+ NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"
```
通过INTEGER属性设置静态变量代表颜色：

```
public static final int DEFAULT_COLOR = 0; //白
public static final int YELLOW_COLOR = 1; //黄
public static final int BLUE_COLOR = 2; //蓝
public static final int GREEN_COLOR = 3; //绿
public static final int RED_COLOR = 4; //红
public static final int BLACK_COLOR = 5;//黑
```
将颜色填充到ListView，可以用SimpleCursorAdapter中的getView，bindView，newView方法来实现，我选择了bindView。自定义一个CursorAdapter继承SimpleCursorAdapter，既能完成cursor读取的数据库内容填充到item，又能将颜色填充：

```
public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 白 255 255 255
         * 黄 247 216 133
         * 蓝 165 202 237
         * 绿 161 214 174
         * 红 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```
在菜单文件editor_options_menu.xml添加更换背景颜色：

```
<item android:id="@+id/menu_color"
    android:title="背景颜色"
    android:showAsAction="always" />
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：

```
case R.id.menu_color:
    changeColor();
    break;
```
添加changeColor（)函数：

```
private final void changeColor() {
    Intent intent = new Intent(null,mUri);
    intent.setClass(NoteEditor.this,NoteColor.class);
    NoteEditor.this.startActivity(intent);
}
```
添加新Activity NoteColor

```
package com.example.android.notepad;

import android.app.Activity;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        //从NoteEditor传入的uri
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
    }
    @Override
    protected void onResume(){
        //执行顺序在onCreate之后
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
        //执行顺序在finish()之后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }

}


```
添加相对应的布局note_color.xml：

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/white"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/yellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/blue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/green"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/red"
        android:onClick="red"/>
</LinearLayout>
```
实现后效果如图

![](https://github.com/c815852517/NotePad_plus/blob/master/app/背景色1.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/背景色2.png)

#### 6.导出笔记至本地
先在editor_options_menu.xml中添加一个导出笔记的选项：

```
<item android:id="@+id/menu_output"
    android:title="导出文件" />
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：

```
//导出笔记选项
case R.id.menu_output:
    outputNote();
    break;
```
添加函数outputNote()：

```
//跳转导出笔记的activity，将uri信息传到新的activity
private final void outputNote() {
    Intent intent = new Intent(null,mUri);
    intent.setClass(NoteEditor.this,OutputText.class);
    NoteEditor.this.startActivity(intent);
}
```
添加新Activity OutputTest：

```
package com.example.android.notepad;

import android.app.Activity;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.os.Environment;
import android.view.View;
import android.widget.EditText;
import android.widget.Toast;

import java.io.File;
import java.io.FileOutputStream;
import java.io.OutputStreamWriter;
import java.io.PrintWriter;

public class OutputText extends Activity {
    //要使用的数据库中笔记的信息
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_NOTE, // 2
            NotePad.Notes.COLUMN_NAME_CREATE_DATE, // 3
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 4
    };
    //读取出的值放入这些变量
    private String TITLE;
    private String NOTE;
    private String CREATE_DATE;
    private String MODIFICATION_DATE;
    //读取该笔记信息
    private Cursor mCursor;
    //导出文件的名字
    private EditText mName;
    //NoteEditor传入的uri，用于从数据库查出该笔记
    private Uri mUri;
    //关于返回与保存按钮的一个特殊标记，返回的话不执行导出，点击按钮才导出
    private boolean flag = false;
    private static final int COLUMN_INDEX_TITLE = 1;
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.output_text);
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
        mName = (EditText) findViewById(R.id.output_name);
    }
    @Override
    protected void onResume(){
        super.onResume();
        if (mCursor != null) {
            // The Cursor was just retrieved, so its index is set to one record *before* the first
            // record retrieved. This moves it to the first record.
            mCursor.moveToFirst();
            //编辑框默认的文件名为标题，可自行更改
            mName.setText(mCursor.getString(COLUMN_INDEX_TITLE));
        }
    }
    @Override
    protected void onPause() {
        super.onPause();
        if (mCursor != null) {
            //从mCursor读取对应值
            TITLE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_TITLE));
            NOTE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_NOTE));
            CREATE_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_CREATE_DATE));
            MODIFICATION_DATE = mCursor.getString(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE));
            //flag在点击导出按钮时会设置为true，执行写文件
            if (flag == true) {
                write();
            }
            flag = false;
        }
    }
    public void OutputOk(View v){
        flag = true;
        finish();
    }
    private void write()
    {
        try
        {
            // 如果手机插入了SD卡，而且应用程序具有访问SD的权限
            if (Environment.getExternalStorageState().equals(
                    Environment.MEDIA_MOUNTED)) {
                // 获取SD卡的目录
                File sdCardDir = Environment.getExternalStorageDirectory();
                //创建文件目录
                File targetFile = new File("/sdcard/Download"+ "/" + mName.getText() + ".txt");
                if(!targetFile.exists()){
                    targetFile.createNewFile();
                }
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + "/sdcard/Download" + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}

```
添加相对应的布局output_text.xml:

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
    <EditText android:id="@+id/output_name"
        android:maxLines="1"
        android:layout_marginTop="2dp"
        android:layout_marginBottom="15dp"
        android:layout_width="wrap_content"
        android:ems="25"
        android:layout_height="wrap_content"
        android:autoText="true"
        android:capitalize="sentences"
        android:scrollHorizontally="true" />
    <Button android:id="@+id/output_ok"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="right"
        android:text="确认导出"
        android:onClick="OutputOk" />
</LinearLayout>
```
在AndroidManifest.xml中加入权限：

```
<!-- 在SD卡中创建与删除文件权限 -->
<uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"
    tools:ignore="ProtectedPermissions" />
<!-- 向SD卡写入数据权限 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
实现效果如图：

![](https://github.com/c815852517/NotePad_plus/blob/master/app/导出笔记1.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/导出笔记2.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/导出笔记3.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/导出笔记4.png)
![](https://github.com/c815852517/NotePad_plus/blob/master/app/导出笔记5.png)
