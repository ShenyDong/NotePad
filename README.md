# NotePad
---
## 原项目
图1：NotePad主界面<br>
![NotePadMain](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/NotePadMain.png)<br>
图2：新建笔记<br>
![NewNoteEdit](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/NewNoteEdit.png)<br>
图3：新建笔记退回主页面<br>
![NewNoteList](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/NewNoteList.png)<br>
图4：进入笔记，编辑标题菜单<br>
![EditTitle](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/EditTitle.png)<br>
图5：编辑标题<br>
![EditTitleDialog](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/EditTileDialog.png)<br>
图6：笔记列表<br>
![moreNotes](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/moreNotes.png)<br>
图7：长点“第二条笔记”，菜单<br>
![menu](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/menu.png)<br>

## 拓展功能
- 笔记列表显示笔记条目的修改时间
- 笔记内容的搜索功能
- UI美化
- 背景更换
- 笔记排序（按颜色、按修改日期）
- 本地导出



## 拓展功能工程演示

-时间戳（图为UI修改后）
![main](https://github.com/ShenyDong/NotePad/blob/master/截图文件/修改完颜色界面.png)<br>
- 笔记列表显示笔记条目的修改时间

首先，找到列表中item的布局：noteslist_item.xml。<br>
在标题的TextView下再加一个时间的TextView。<br>
```
    <!--添加显示时间的TextView-->
    <TextView
        android:id="@+id/text1_time"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:textAppearance="?android:attr/textAppearanceSmall"
        android:paddingLeft="5dip"
        android:textColor="@color/colorBlack"/>
</LinearLayout>
```
在NotePadProvider.java中可以看出，NotePad数据库已经存在时间信息。<br>到NotesList.java文件中将数据装填完善<br>
```
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
    };
Cursor cursor = managedQuery(
            getIntent().getData(),            // Use the default content URI for the provider.
            PROJECTION,                       // Return the note ID and title for each note.
            null,                             // No where clause, return all records.
            null,                             // No where clause, therefore no where column values.
            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
SimpleCursorAdapter adapter
    = new SimpleCursorAdapter(
            this, // The Context for the ListView
            R.layout.noteslist_item, // Points to the XML for a list item
            cursor,   // The cursor to get items from
            dataColumns,
            viewIDs
    );
// Sets the ListView's adapter to be the cursor adapter that was just created.
setListAdapter(adapter);
 private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR, 
    };
```
Cursor不变，在dataColumns，viewIDs中补充时间部分：<br>
```
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE } ;
int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
```
时间戳实现<br>
```
Long now = Long.valueOf(System.currentTimeMillis());
Date date = new Date(now);
SimpleDateFormat format = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
String dateTime = format.format(date);
```







- 笔记搜索

搜索结果显示<br>

![searchmenu](https://github.com/ShenyDong/NotePad/blob/master/截图文件/搜索.png)

先修改布局。找到菜单的xml文件，list_options_menu.xml，添加一个搜索的item<br>
```
<item
    android:id="@+id/menu_search"
    android:title="@string/menu_search"
    android:icon="@android:drawable/ic_search_category_default"
    android:showAsAction="always">
</item>
```
新建一个名为NoteSearch的activity。所以可以模仿NotesList的，在layout中新建布局文件note_search_list.xml：<br>
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="输入搜索内容..."
        android:layout_alignParentTop="true">
    </SearchView>
    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
    </ListView>
</LinearLayout>
```
在NotesList中找到onOptionsItemSelected方法，在switch中添加搜索的case语句:<br>
```
    case R.id.menu_search:
    intent.setClass(NotesList.this,NoteSearch.class);
    NotesList.this.startActivity(intent);
    return true;
```


要动态地显示搜索结果，就要对SearchView文本变化设置监听<br>
```
public class NoteSearch extends ListActivity  implements SearchView.OnQueryTextListener {
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //扩展 显示时间 颜色
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 2
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search_list);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        SearchView searchview = (SearchView)findViewById(R.id.search_view);
        //为查询文本框注册监听器
        searchview.setOnQueryTextListener(NoteSearch.this);  
    }
    @Override
    public boolean onQueryTextSubmit(String query) {
        return false;
    }
    @Override
    public boolean onQueryTextChange(String newText) {
        String selection = NotePad.Notes.COLUMN_NAME_TITLE + " Like ? ";
        String[] selectionArgs = { "%"+newText+"%" };
        Cursor cursor = managedQuery(
                getIntent().getData(),            // Use the default content URI for the provider.
                PROJECTION,                       // Return the note ID and title for each note. and modifcation date
                selection,                        // 条件左边
                selectionArgs,                    // 条件右边
                NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
        );
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,  NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE };
        int[] viewIDs = { android.R.id.text1 , R.id.text1_time };
        MyCursorAdapter adapter = new MyCursorAdapter(
                this,
                R.layout.noteslist_item,
                cursor,
                dataColumns,
                viewIDs
        );
        setListAdapter(adapter);
        return true;
    }
    @Override
    protected void onListItemClick(ListView l, View v, int position, long id) {
        // Constructs a new URI from the incoming URI and the row ID
        Uri uri = ContentUris.withAppendedId(getIntent().getData(), id);
        // Gets the action from the incoming Intent
        String action = getIntent().getAction();
        // Handles requests for note data
        if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
            // Sets the result to return to the component that called this Activity. The
            // result contains the new URI
            setResult(RESULT_OK, new Intent().setData(uri));
        } else {
            // Sends out an Intent to start an Activity that can handle ACTION_EDIT. The
            // Intent's data is the note ID URI. The effect is to call NoteEdit.
            startActivity(new Intent(Intent.ACTION_EDIT, uri));
        }
    }
}

```



- UI美化

首页界面：<br>
![main](https://github.com/ShenyDong/NotePad/blob/master/截图文件/界面.png)

![main](https://github.com/ShenyDong/NotePad/blob/master/截图文件/修改完颜色界面.png）

为了更好的现实笔记颜色，把黑色换成白色，style.xml文件中重写theme，为首页添加背景，重写界面主题和弹框主题。
```
      <style name="BaseTheme" parent="MyTheme">
        <item name="android:windowBackground">@drawable/main_background</item>
    </style>

    <style name="App" parent="android:Theme.Holo">
    <item name="android:layout_width">match_parent</item>
    <item name="android:layout_height">50dp</item>
    <item name="android:background">@color/colorPink</item>
    </style>

    <style name="AppDialog" parent="android:Theme.Dialog">
        <item name="android:layout_width">match_parent</item>
        <item name="android:layout_height">50dp</item>
        <item name="android:textColor">#ffffff</item>
        <item name="android:background">@color/colorPink</item>
    </style>
```
在AndroidManifest.xml中NotesList的Activity中添加：<br>
```
        <activity android:name="NotesList" android:label="@string/title_notes_list"
            android:theme="@style/BaseTheme">
```





- 更改背景颜色显示

修改笔记一的背景色：<br>
![changecolornote](https://github.com/ShenyDong/NotePad/blob/master/截图文件/选择颜色.png)<br>
选择菜单上的调色盘为的图标<br>
选择蓝色：<br>
![background](https://github.com/ShenyDong/NotePad/blob/master/截图文件/蓝色.png)<br>

NoteEditor类

```
  private static final String[] PROJECTION =
        new String[] {
            NotePad.Notes._ID,
            NotePad.Notes.COLUMN_NAME_TITLE,
            NotePad.Notes.COLUMN_NAME_NOTE,
            NotePad.Notes.COLUMN_NAME_BACK_COLOR
    };
```
```
int x = mCursor.getInt(mCursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
    switch (x){
        case NotePad.Notes.DEFAULT_COLOR:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
        case NotePad.Notes.YELLOW_COLOR:
            mText.setBackgroundColor(Color.rgb(247, 216, 133));
            break;
        case NotePad.Notes.BLUE_COLOR:
            mText.setBackgroundColor(Color.rgb(165, 202, 237));
            break;
        case NotePad.Notes.GREEN_COLOR:
            mText.setBackgroundColor(Color.rgb(161, 214, 174));
            break;
        case NotePad.Notes.RED_COLOR:
            mText.setBackgroundColor(Color.rgb(244, 149, 133));
            break;
        default:
            mText.setBackgroundColor(Color.rgb(255, 255, 255));
            break;
    }
```
新建布局note_color.xml，垂直线性布局放置5个ImageButton，并创建NoteColor的Acitvity，用来选择颜色。在后期UI美化过程中，将按钮设置成了圆形。
在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式：<br>
```
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="65dp"
        android:layout_weight="1"
        android:background="@drawable/white"
        android:onClick="white"/>

    <ImageButton
        android:id="@+id/color_Purple"
        android:layout_width="0dp"
        android:layout_height="65dp"
        android:layout_weight="1"
        android:background="@drawable/purple"
        android:onClick="purple"/>

    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dip"
        android:layout_height="65dp"
        android:layout_weight="1"
        android:background="@drawable/circle"
        android:onClick="blue"/>

    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="65dp"
        android:layout_weight="1"
        android:background="@drawable/green"
        android:onClick="green"/>

    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="65dp"
        android:layout_weight="1"
        android:background="@drawable/red"
        android:onClick="red"/>
```
NoteColor类：
```
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



- 导出笔记

点击隐藏的菜单，选择本地导出：<br>

 ![outputmenu](https://github.com/ShenyDong/NotePad/blob/master/截图文件/本地导出.png)<br>
 弹出弹框，输入导出标题
 
 ![outputmenu](https://github.com/ShenyDong/NotePad/blob/master/截图文件/导出名字.png)<br>

先在菜单文件中添加一个导出笔记的选项，editor_options_menu.xml：<br>
```
<item android:id="@+id/menu_output"
    android:title="@string/menu_output" />
```
在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：<br>
```
//导出笔记选项
   case R.id.menu_output:
        outputNote();
        break;
```
在NoteEditor中添加函数outputNote()：<br>
```
//跳转导出笔记的activity，将uri信息传到新的activity
    private final void outputNote() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,OutputText.class);
        NoteEditor.this.startActivity(intent);
    }
```
新建布局output_text.xml：<br>

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
        android:text="@string/output_ok"
        android:onClick="OutputOk" />
</LinearLayout>
```
```
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
                File targetFile = new File(sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt");
                //写文件
                PrintWriter ps = new PrintWriter(new OutputStreamWriter(new FileOutputStream(targetFile), "UTF-8"));
                ps.println(TITLE);
                ps.println(NOTE);
                ps.println("创建时间：" + CREATE_DATE);
                ps.println("最后一次修改时间：" + MODIFICATION_DATE);
                ps.close();
                Toast.makeText(this, "保存成功,保存位置：" + sdCardDir.getCanonicalPath() + "/" + mName.getText() + ".txt", Toast.LENGTH_LONG).show();
            }
        }
        catch (Exception e)
        {
            e.printStackTrace();
        }
    }
}
```
在AndroidManifest.xml中将这个Acitvity主题定义为对话框样式，并且加入权限：<br>
```
<!--添加导出activity-->
        <activity android:name="OutputText"
            android:label="@string/output_name"
            android:theme="@android:style/Theme.Holo.Dialog"
            android:windowSoftInputMode="stateVisible">
        </activity>
```
```
 <!-- 在SD卡中创建与删除文件权限 -->
    <uses-permission android:name="android.permission.MOUNT_UNMOUNT_FILESYSTEMS"/>
    <!-- 向SD卡写入数据权限 -->
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```



- 笔记排序

创建时间排序：<br>
![createtime](https://github.com/ShenyDong/NotePad/blob/master/截图文件/按创建时间.png)<br>
修改时间排序：<br>
![modifytime](https://github.com/ShenyDong/NotePad/blob/master/截图文件/修改完颜色界面.png)<br>
颜色排序：<br>
![colorsort](https://github.com/ShenyDong/NotePad/blob/master/截图文件/按颜色.png)<br>
在菜单文件list_options_menu.xml中添加：<br>

    <item
        android:id="@+id/menu_sort"
        android:title="@string/menu_sort"
        android:icon="@drawable/ic_insert"
        android:showAsAction="always" >


- 扩展后的目录结构

![dirstructure1](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/dirstructure1.png)<br>
![dirstructure2](https://raw.githubusercontent.com/douerza/picture/master/NotePadPic/dirstructure2.png)<br>
