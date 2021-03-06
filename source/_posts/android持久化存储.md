---
title: android持久化存储
date: 2018-01-03 14:57:26
tags: android
---

# 内部存储（internalStorage）

data文件夹就是我们常说的内部存储，其中有两个文件夹需要关注，一个是app文件夹，一个是data文件夹。

## app文件夹

app文件夹里存放着我们所有安装的app的apk文件。

## data文件夹

这个文件夹里面都是一些包名，打开这些包名会看到：

1. data/data/包名/shared_prefs
2. data/data/包名/databases
3. data/data/包名/files
4. data/data/包名/cache

sharedPreferenced的时候，数据持久化存储于本地，其实就是存在这个文件中的xml文件里面。
数据库文件就是存储于databases文件夹中。
普通数据存储在files中。
缓存文件存储在cache文件夹中。


# 外部存储（externalStorage）

外部存储是我们平时操作最多的，外部存储一般就是我们看到的storage文件夹，当然也有mnt文件夹，不同厂家有可能不一样。
一般来讲storage文件夹中有一个sdcard文件夹，这个文件夹中的文件又分为两类，一类是公有目录，一类是私有目录。公有目录又分为九大类，比如说DCIM、DOWNLOAD等这种系统为我们创建的文件夹。私有目录就是android这种文件夹，这个文件夹打开之后里面有一个data文件夹，打开这个data文件夹，里面有许多包名组成的文件夹。

# 存储操作

## 文件存储

文件存储的所有文件默认放在/data/data/<packagename>/file/目录下

### 文件写入

```
public void save(String inputText){
	FIleOutputStream out = null;
	BufferedWriter writer = null;
	try{
		out = openFileOutput("data", Context.MODE_PRIVATE);
		writer = new BufferedWriter(new OutputStreamWriter(out));
		writer.write(inputText);
	}catch(FileNotFoundException e){
		e.printStackTrace();
	}catch(IOException e){
		e.printStackTrace();
	}finally{
		if(writer != null){
			try{
				writer.close();
			}catch(IOException e){
				e.printStackTrace();
			}
		}
	}
}
```

### 文件读取

```
public String load(){
	FileInputStream in = null;
	BufferedReader reader = null;
	StringBuilder content = new StringBuilder();
	try{
		in = openFileInput("data");
		reader = new BufferedReader(new InputStreamReader(in));
		String line = "";
		while((line = reader.readLine()) != null){
			content.append(line);
		}
	}catch(FileNotFoundException e){
		e.printStackTrace();
	}catch(IOException e){
		e.printStackTrace();
	}finally{
		if(reader != null){
			try{
				reader.close();
			}catch(IOException e){
				e.printStackTrace();
			}
		}
	}
	return content.toString();
}
```

## SharePreferences

Sharepreference默认放在/data/data/<packagename>/file/目录下

### SharedPreferences写入

```
	SharedPreferences.Editor editor = getSharedPreferences("data", MODE_PRIVATE).edit();
	editor.putString("et_inputText", "sharePreferences test");
	editor.commit();
```

### SharedPreferences读取

```
	SharedPreferences sp = getSharedPreferences("data", MODE_PRIVATE);
	String input = sp.getString("et_inputText", "请输入用户名");//第二个参数是为空的默认信息
```

## SQLite数据库存储

sqlite文件默认放在/data/data/<packagename>/datanases/目录下

```
public class MyDataBaseHelper extends SQLiteOpenHelper{
	private static final String CREATE_BOOK = "create table Book(id integer primary key autoincrement, author text, price real, pages integer, name text)";
	private static final String CREATE_CATEGORY = "create table Category(id integer primary key autoincrement, category_name text, category_code integer)";
	private Context mContext;
	public MyDataBaseHelper(Context context, String name, SQLiteDatabase.CursorFactory factory, int version){
		super(context, name, factory, version);
		mContext = context;
	}
	@Override
	public void onCreate(SQLiteDatabase db){
		db.execSQL(CREATE_BOOK);
		db.execSQL(CREATE_CATEGORY);
		Toast.makeText(mContext, "create succeed", Toast.LENGTH_SHORT).show();
	}
	@Override
	public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion){
		db.execSQL("drop table if exists Book");
		db.execSQL("drop table if exists Category");
		onCreate(db);
	}
}
```

### 创建数据库

```
	dbHelper = new MyDataBaseHelper(this, "BookStore.db", null, 2);
```


### 插入数据

```
	SQLiteDatabase db = dbHelper.getWritableDatabase();
	ContentValues values = new ContentValues();
	values.put("name", "Effective Java");
	values.put("author", "Joshua Bloch");
	values.put("pages", 454);
	values.put("price", 16.96);
	db.insert("Book", null, values);
```

### 更新数据

```
	ContentValues values = new ContentValues();
	values.put("price", 198.00);
	SQLiteDatabase db = dbHelper.getReadableDatabase();
	db.updata("Book", values, "name=?",new String[]{'Android Programme'});
```

### 删除行

```
	SQLiteDatabase db = dbHelper.getWritableDatabase();
	db.delete("Book", "pages > ?", new String[]{"500"});
```