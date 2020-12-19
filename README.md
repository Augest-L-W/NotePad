# 期中项目NotePad
·此README文件用于介绍本次期中项目实验。

若无法显示图片则点击：[Notepad图片](https://blog.csdn.net/weixin_45738069/article/details/111412756)


·首先是进行工程的导入，但在导入工程时发生错误，通过记事本更改NotePad-master工程的，build.gradle(project)、build.gradle(app)、gradle-wrapper.properties文件为能正常编译的文件内容，并去掉AndroidManifest.xml中的语句再进行工程的导入，最终编译成功。
### 时间戳显示
第一步：在noteslist_item布局文件中增加一个id为text2的TextView用于显示记事本的修改时间，其代码如下：

```
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text2"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
    />

```

第二步：修改NotePadProvide类中的READ_NOTE_PROJECTION字符串数组，添加创建、修改时间的映射，修改后的代码如下：

```
private static final String[] READ_NOTE_PROJECTION = new String[] {
        NotePad.Notes._ID,               // Projection position 0, the note's id
        NotePad.Notes.COLUMN_NAME_NOTE,  // Projection position 1, the note's content
        NotePad.Notes.COLUMN_NAME_TITLE, // Projection position 2, the note's title
        NotePad.Notes.COLUMN_NAME_CREATE_DATE,//@ Projection position 3, the note's create time
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//@ Projection position 4, the note's modification time
};
```

第三步：修改NotesList类对PROJECTION数组增加修改时间映射，在oncreate方法中实现适配器的组装增加获取到的修改时间与第一步新增的text2之间的装配，此时显示的时间为毫秒单位的当前时间；修改代码如下：

```
private static final String[] PROJECTION = new String[] {
        NotePad.Notes._ID, // 0
        NotePad.Notes.COLUMN_NAME_TITLE, // 1
        NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//增加修改时间的映射
};
//-----------------------------------------------------------------------------
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
int[] viewIDs = { android.R.id.text1,android.R.id.text2 };
```

第四步：修改NoteEditor类中的update方法，先获取到系统的时间再将该时间转化为年月日-时分秒的字符串格式，再将该时间写入修改时间数据中去，从而使得最终显示时间为年月日-时分秒形式；
实现代码如下：

```
// 进行时间获取及格式的转换，最终将得到的时间赋值到数据表中的modification时间
ContentValues values = new ContentValues();
SimpleDateFormat dateformat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String dateStr = dateformat.format(System.currentTimeMillis());
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateStr);
```

最后：再次修改noteslist_item.xml布局文件，新增ImageView，使其显示为左边小图标、右边标题与修改时间的显示形式，更加美观
整个修改后的noteslist_item.xml实现代码如下：

```
<LinearLayout xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="horizontal">

    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/noteimage"
        android:src="@drawable/app_notes"
        >
    </ImageView>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical">
        <TextView xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@android:id/text1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:gravity="center_vertical"
            android:paddingLeft="5dip"
            android:singleLine="true"
            />
        <TextView xmlns:android="http://schemas.android.com/apk/res/android"
            android:id="@android:id/text2"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center_vertical"
            android:paddingLeft="5dip"
            android:singleLine="true"
            />
        </LinearLayout>

</LinearLayout>
```

运行后效果图如下：


![image](https://github.com/Augest-L-W/picture/blob/master/%E5%85%A8%E7%A7%B0%E6%90%9C%E7%B4%A2%E6%88%AA%E5%9B%BE.png)

### 搜索功能
第一步：在menu文件夹下创建main_menu.xml文件来配置菜单栏的搜索框及样式，其中搜索小图标为自己做的小图标，即为存放在drawable文件夹下的search.PNG文件,配置搜索框代码如下：
```
<item
    android:id="@+id/search"
    android:icon="@drawable/search"
    android:title="Search"
    android:actionViewClass="android.widget.SearchView"
    android:showAsAction="always"
></item>

```

第二步：在NoteList类中的onCreateOptionsMenu方法将第一步定义的菜单栏资源对象进行获取并实例化为SearchView对象进行搜索框相关属性的设置以及提交、输入值的响应事件进行监听与处理和设置点击×退出搜索框监听与响应事件，最终完成搜索框的模糊查询，退出会重新搜索当前存在的笔记记录功能，完整的实现了搜索功能。实现上述功能的代码如下：

```
public boolean onCreateOptionsMenu(Menu menu) {
    // Inflate menu from XML resource
    MenuInflater inflater = getMenuInflater();
    inflater.inflate(R.menu.list_options_menu, menu);

    // Generate any additional actions that can be performed on the
    // overall list.  In a normal install, there are no additional
    // actions found here, but this allows other applications to extend
    // our menu with their own actions.
    Intent intent = new Intent(null, getIntent().getData());
    intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
    menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
            new ComponentName(this, NotesList.class), null, intent, 0, null);

    //获取菜单搜索宽样式，并通过SearchView对象进行搜索框相关属性的设置
    // 以及提交、输入值的响应事件进行监听与处理
    getMenuInflater().inflate(R.menu.main_menu,menu);
    SearchView mysearchview=(SearchView) menu.findItem(R.id.search).getActionView();
    mysearchview.setQueryHint("搜索");//设置提示信息
    mysearchview.setOnQueryTextListener(new SearchView.OnQueryTextListener(){
        @Override
        //当提交搜索框内容后执行的方法
        public boolean onQueryTextSubmit(String query) {
            search(query);//查询数据库中匹配该结果的数据

            return false;
        }

        @Override
        //当搜索框内内容改变时执行的方法
        public boolean onQueryTextChange(String query) {

            return false;
        }

    });
    //设置点击×退出搜索框监听与响应事件
    mysearchview.setOnCloseListener(new SearchView.OnCloseListener() {
        //添加点击关闭的监听事件，使得点击关闭后搜索出现有的所有笔记
        @Override
        public boolean onClose() {
            refresh();
            return false;
        }
    });


    return super.onCreateOptionsMenu(menu);
}

//对提交的信息进行模糊查询返回查询到的结果，该编码实际为再次的有条件获取数据库信息并用适配器进行适配展现出来
void search(String key)
{
    Intent intent = getIntent();


    if (intent.getData() == null) {
        intent.setData(NotePad.Notes.CONTENT_URI);
    }

    getListView().setOnCreateContextMenuListener(this);

    //通过设置查询条件，达到模糊查询的实现
    String selection=NotePad.Notes.COLUMN_NAME_TITLE+" LIKE ?";
    String[] selectionArgs={"%"+key+"%"};

    Cursor cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            selection,
            selectionArgs,
            NotePad.Notes.DEFAULT_SORT_ORDER
    );

    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

    int[] viewIDs = { android.R.id.text1,android.R.id.text2 };

    SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
            this,                             // The Context for the ListView
            R.layout.noteslist_item,          // Points to the XML for a list item
            cursor,                           // The cursor to get items from
            dataColumns,
            viewIDs
    );
    setListAdapter(adapter);

}
//再次的获取数据库中所有笔记信息并用适配器进行适配展现出来（实质为再次执行OnCreate中的部分代码）
void refresh(){
    Intent intent = getIntent();


    if (intent.getData() == null) {
        intent.setData(NotePad.Notes.CONTENT_URI);
    }

    getListView().setOnCreateContextMenuListener(this);

    String selection=null;
    String[] selectionArgs=null;

    Cursor cursor = managedQuery(
            getIntent().getData(),
            PROJECTION,
            selection,
            selectionArgs,
            NotePad.Notes.DEFAULT_SORT_ORDER
    );

    String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;

    int[] viewIDs = { android.R.id.text1,android.R.id.text2 };

    SimpleCursorAdapter adapter
            = new SimpleCursorAdapter(
            this,                             // The Context for the ListView
            R.layout.noteslist_item,          // Points to the XML for a list item
            cursor,                           // The cursor to get items from
            dataColumns,
            viewIDs
    );
    setListAdapter(adapter);

}
```

运行截图1（全称搜索）：

![image](https://github.com/Augest-L-W/picture/blob/master/%E5%85%A8%E7%A7%B0%E6%90%9C%E7%B4%A2%E6%88%AA%E5%9B%BE.png)

运行截图2（关键词搜索）：

![image](https://github.com/Augest-L-W/picture/blob/master/%E5%85%B3%E9%94%AE%E8%AF%8D%E6%90%9C%E7%B4%A2%E6%88%AA%E5%9B%BE.png)





···进行点击搜索框内的×号，退出搜索框并显示已有的笔记内容操作实现要点，对点击×退出搜索框操作进行事件监听和响应方法，在响应方法中使用oncreate方法中进行数据查询与数据装配的方法进行搜索内容的查询即可，即selection条件与selectionArgs参数条件都为null，具体代码为

```
//设置点击×退出搜索框监听与响应事件
mysearchview.setOnCloseListener(new SearchView.OnCloseListener() {
    //添加点击关闭的监听事件，使得点击关闭后搜索出现有的所有笔记
    @Override
    public boolean onClose() {
        refresh();
        return false;
    }
});

String selection=null;
String[] selectionArgs=null;
```

退出截图1（第一次点击）

![image](https://github.com/Augest-L-W/picture/blob/master/%E7%AC%AC%E4%B8%80%E6%AC%A1%E7%82%B9%E5%87%BB%E6%88%AA%E5%9B%BE.png)

退出截图2（第二次点击）

![image](https://github.com/Augest-L-W/picture/blob/master/%E9%A6%96%E9%A1%B5%E6%88%AA%E5%9B%BE.png)


## 扩展功能1（优化删除功能）
一、部分删除，增加一个提示功能避免误点删除有用信息。点击删除图标后会有一个弹窗提示，点击OK删除成功，点击CANCEL则撤销删除操作；实现代码如下：

```
@Override
    public boolean onOptionsItemSelected(MenuItem item){
        switch (item.getItemId()){
            case R.id.delete:
                new AlertDialog.Builder(EditActivity.this)
                        .setMessage("您确定删除吗？")
                        .setPositiveButton(android.R.string.yes, new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                if (openMode == 4){ // new note
                                    intent.putExtra("mode", -1);
                                    setResult(RESULT_OK, intent);
                                }
                                else { // existing note
                                    intent.putExtra("mode", 2);
                                    intent.putExtra("id", id);
                                    setResult(RESULT_OK, intent);
                                }
                                finish();
                            }
                        }).setNegativeButton(android.R.string.no, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                }).create().show();
                break;
        }
        return super.onOptionsItemSelected(item);
    }
```

运行截图：

![image](https://github.com/Augest-L-W/picture/blob/master/%E9%83%A8%E5%88%86%E5%88%A0%E9%99%A4%E6%88%AA%E5%9B%BE.png)

二、增加全部删除功能，方便管理。在首页点击删除图标后有一个弹窗提示，点击OK则删除全部便签，点击CANCEL则撤销删除操作；实现代码如下：

```
@Override
    public boolean onOptionsItemSelected(MenuItem item){
        switch (item.getItemId()){
            case R.id.menu_clear:
                new AlertDialog.Builder(MainActivity.this)
                        .setMessage("您确定删除全部吗？")
                        .setPositiveButton(android.R.string.yes, new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialog, int which) {
                                dbHelper = new NoteDatabase(context);
                                SQLiteDatabase db = dbHelper.getWritableDatabase();
                                db.delete("notes", null, null);
                                db.execSQL("update sqlite_sequence set seq=0 where name='notes'");
                                refreshListView();
                            }
                        }).setNegativeButton(android.R.string.no, new DialogInterface.OnClickListener() {
                    @Override
                    public void onClick(DialogInterface dialog, int which) {
                        dialog.dismiss();
                    }
                }).create().show();
                break;
 
        }
        return super.onOptionsItemSelected(item);
    }
```

运行截图：

![image](https://github.com/Augest-L-W/picture/blob/master/%E5%85%A8%E9%83%A8%E5%88%A0%E9%99%A4%E6%88%AA%E5%9B%BE.png)



### 第二个扩展小功能：每个便签左侧增加一个小图标
在布局文件noteslist_item.xml里添加一个ImageView布局，添加代码如下：

```
    <ImageView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/noteimage"
        android:src="@drawable/app_notes"
        >
    </ImageView>
```

截图：
![image](https://github.com/Augest-L-W/picture/blob/master/%E9%A6%96%E9%A1%B5%E6%88%AA%E5%9B%BE.png)

