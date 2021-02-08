# 跨进程示例

## AIDL 服务端
	
	aidl的文件在main文件下，aidl的文件分为两种类型，bean文件类型和interface类型

### 1.bean文件类型（注意类的位置在java（Book.java）和aidl(Book.aidl)的下的位置要一致）
	-Book.java
				package com.timaimee.demohband.aidldemo;
				import android.os.Parcel;
				import android.os.Parcelable;
				
				public class Book implements Parcelable {
				    private String bookName;
				    private int bookId;
				
				    public Book(String bookName,int bookId) {
				        this.bookName = bookName;
				        this.bookId = bookId;
				    }
				
				    /**
				     * 从binder中读取
				     *
				     * @param in
				     */
				    protected Book(Parcel in) {
				        bookId = in.readInt();
				        bookName = in.readString();
				    }
				
				    public static final Creator<Book> CREATOR = new Creator<Book>() {
				        @Override
				        public Book createFromParcel(Parcel in) {
				            return new Book(in);
				        }
				
				        @Override
				        public Book[] newArray(int size) {
				            return new Book[size];
				        }
				    };
				
				    @Override
				    public int describeContents() {
				        return 0;
				    }
				
				    /**
				     * 写入binder
				     *
				     * @param dest
				     * @param flags
				     */
				    @Override
				    public void writeToParcel(Parcel dest, int flags) {
				        dest.writeInt(bookId);
				        dest.writeString(bookName);
				    }
				
				    @Override
				    public String toString() {
				        return "Book{" +
				                "bookName='" + bookName + '\'' +
				                ", bookId=" + bookId +
				                '}';
				    }
				}

		Book.aidl
			package com.timaimee.demohband.aidldemo;
			// Declare any non-default types here with import statements
			parcelable Book;

### 2.interface类型
	
	-IBookManager.aidl
		//声明可调用的方法的接口，要在service中实现的
		package com.timaimee.demohband.aidldemo;
		import com.timaimee.demohband.aidldemo.Book;
		import com.timaimee.demohband.aidldemo.IOnNewBookArrivedListener;
		
		interface IBookManager {
		    List<Book> getBookList(); // 返回书籍列表
		    void addBook(in Book book); // 添加书籍
		    void registerListener(IOnNewBookArrivedListener listener); // 注册接口
		    void unregisterListener(IOnNewBookArrivedListener listener); // 注册接口
		}

	-IOnNewBookArrivedListener.aidl
		//声明回调接口，客户端里面进行注册，服务端里面调用
		package com.timaimee.demohband.aidldemo;
		import com.timaimee.demohband.aidldemo.Book;
		
		// Declare any non-default types here with import statements
		
		interface IOnNewBookArrivedListener  {
		    void onNewBookArrived(in Book newBook);
		}

### 实现service

	-BookManagerService.java

	package com.timaimee.demohband.aidldemo;
	
	import android.app.Service;
	import android.content.Intent;
	import android.os.Binder;
	import android.os.Handler;
	import android.os.IBinder;
	import android.os.Message;
	import android.os.RemoteCallbackList;
	import android.os.RemoteException;
	import android.util.Log;
	
	import androidx.annotation.NonNull;
	import androidx.annotation.Nullable;
	
	import java.util.List;
	import java.util.concurrent.CopyOnWriteArrayList;
	import java.util.concurrent.atomic.AtomicBoolean;
	
	public class BookManagerService extends Service {
	    private static final String TAG = BookManagerService.class.getSimpleName();
	    private CopyOnWriteArrayList<Book> mBookList = new CopyOnWriteArrayList<>();
	    //存储注册监听客户端集合
	    private RemoteCallbackList<IOnNewBookArrivedListener> mListenerList = new RemoteCallbackList<>();
	    private Handler mHandler = new Handler() {
	        @Override
	        public void handleMessage(@NonNull Message msg) {
	            super.handleMessage(msg);
	            int bookId = 1 + mBookList.size();
	            Book newBook = new Book("新书#" + bookId, bookId);
	            try {
	                onNewBookArrived(newBook);
	            } catch (RemoteException e) {
	                e.printStackTrace();
	            }
	        }
	    };
	
	    private void onNewBookArrived(Book book) throws RemoteException {
	        mBookList.add(book);
	        Log.e(TAG, "发送通知的数量: " + mBookList.size());
	        int num = mListenerList.beginBroadcast();
	        for (int i = 0; i < num; ++i) {
	            IOnNewBookArrivedListener listener = mListenerList.getBroadcastItem(i);
	            Log.e(TAG, "发送通知: " + listener.toString());
	            listener.onNewBookArrived(book);
	        }
	        mListenerList.finishBroadcast();
	    }
	
	
	    Binder mBinder = new IBookManager.Stub() {
	        @Override
	        public List<Book> getBookList() throws RemoteException {
	            return mBookList;
	        }
	
	        @Override
	        public void addBook(Book book) throws RemoteException {
	            onNewBookArrived(book);
	        }
	
	        @Override
	        public void registerListener(IOnNewBookArrivedListener listener) throws RemoteException {
	            mListenerList.register(listener);
	            int num = mListenerList.beginBroadcast();
	            mListenerList.finishBroadcast();
	        }
	
	        @Override
	        public void unregisterListener(IOnNewBookArrivedListener listener) throws RemoteException {
	            mListenerList.unregister(listener);
	            mListenerList.finishBroadcast();
	        }
	    };
	
	
	    @Nullable
	    @Override
	    public IBinder onBind(Intent intent) {
	        return mBinder;
	    }
	
	    @Override
	    public void onCreate() {
	        super.onCreate();
	        mBookList.add(new Book("ANDROID BOOK", 1));
	        mBookList.add(new Book("IOS BOOK", 2));
	    }
	
	    @Override
	    public void onDestroy() {
	        super.onDestroy();
	    }
	
	}



### Manifest.xml注册

如果是只在同一个app调用演示，android:process表示是在新的进程中运行

        <service
            android:name="com.timaimee.demohband.aidldemo.BookManagerService"
            android:process=":remote">
        </service>
		绑定调用的方式如下：
			    Intent intent = new Intent(this, BookManagerService.class);
        		bindService(intent, mConnection, Context.BIND_AUTO_CREATE);

方便其他app来调用，android:exported表示是可以被另一个Application的组件启动

        <service
            android:name="com.timaimee.demohband.aidldemo.BookManagerService"
            android:exported=true>
            <intent-filter>
                <action android:name="lijinliang" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>
		绑定调用的方式如下：
				Intent intent = new Intent();
        		intent.setPackage("com.timaimee.demohband");
        		intent.setAction("lijinliang");
        		bindService(intent, mConnection, Context.BIND_AUTO_CREATE）
		实测中发现 android:exported=true换成android:process=":remote"也是可以绑定成功的

	
## AIDL 客户端(同一个app)

	-AidlActivity.java
	
	package com.timaimee.demohband;
	
	import android.app.Activity;
	import android.content.ComponentName;
	import android.content.Context;
	import android.content.Intent;
	import android.content.ServiceConnection;
	import android.os.AsyncTask;
	import android.os.Bundle;
	import android.os.Handler;
	import android.os.IBinder;
	import android.os.Message;
	import android.os.RemoteException;
	import android.util.Log;
	import android.view.View;
	import android.widget.TextView;
	import android.widget.Toast;
	
	import com.timaimee.demohband.aidldemo.Book;
	import com.timaimee.demohband.aidldemo.BookManagerService;
	import com.timaimee.demohband.aidldemo.IBookManager;
	import com.timaimee.demohband.aidldemo.IOnNewBookArrivedListener;
	import com.timaimee.demohband.proxy.BpTestListner;
	import com.timaimee.demohband.proxy.EcgTestListner;
	import com.timaimee.demohband.proxy.HeartTestListner;
	import com.timaimee.demohband.proxy.IStartListenter;
	import com.timaimee.demohband.proxy.ITestListenter;
	import com.timaimee.demohband.proxy.ProxyUtil;
	
	import java.util.List;
	
	import butterknife.ButterKnife;
	import butterknife.OnClick;
	
	public class AidlActivity extends Activity {
	    private String TAG = AidlActivity.class.getSimpleName();
	
	    private static final int MESSAGE_NEW_BOOK_ARRIVED = 1;
	
	    private TextView mTvBookList;
	
	    private IBookManager mRemoteBookManager;
	
	    private IOnNewBookArrivedListener mOnNewBookArrivedListener = new IOnNewBookArrivedListener.Stub() {
	        @Override
	        public void onNewBookArrived(Book newBook) throws RemoteException {
	            Log.e(TAG, "onNewBookArrived");
	            mHandler.obtainMessage(MESSAGE_NEW_BOOK_ARRIVED, newBook).sendToTarget();
	        }
	    };
	
	    private Handler mHandler = new Handler() {
	        @Override
	        public void handleMessage(Message msg) {
	            switch (msg.what) {
	                case MESSAGE_NEW_BOOK_ARRIVED:
	                    Log.e(TAG, "收到的新书: " + msg.obj);
	                    new BookListAsyncTask().execute();
	                    break;
	                default:
	                    super.handleMessage(msg);
	                    break;
	            }
	        }
	    };
	
	    private ServiceConnection mConnection = new ServiceConnection() {
	
	        @Override
	        public void onServiceConnected(ComponentName name, IBinder service) {
	            IBookManager bookManager = IBookManager.Stub.asInterface(service);
	            mRemoteBookManager = bookManager;
	            try {
	                bookManager.registerListener(mOnNewBookArrivedListener);
	            } catch (RemoteException e) {
	                e.printStackTrace();
	            }
	        }
	
	        @Override
	        public void onServiceDisconnected(ComponentName name) {
	            mRemoteBookManager = null;
	            Log.e(TAG, "绑定结束");
	        }
	    };
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_aidl);
	        mTvBookList = (TextView) findViewById(R.id.tv_book_list);
	        ButterKnife.bind(this);
	    }
	
	
	    /**
	     * 绑定服务按钮的点击事件
	     */
	    @OnClick(R.id.bind_aidl)
	    public void bindService() {
	        Intent intent = new Intent(this, BookManagerService.class);
	        bindService(intent, mConnection, Context.BIND_AUTO_CREATE);
	    }
	
	    @OnClick(R.id.add_book)
	    public void add_book() {
	        Book newBook = new Book("学姐的故事", 3);
	        try {
	            mRemoteBookManager.addBook(newBook);
	        } catch (RemoteException e) {
	            e.printStackTrace();
	        }
	//        new BookListAsyncTask().execute();
	    }
	
	    /**
	     * 获取图书数量的异步线程, 直接绑定点击事件
	     *
	     * @param view 视图
	     */
	    public void getBookList(View view) {
	        new BookNumAsyncTask().execute();
	        Toast.makeText(getApplicationContext(), "正在获取中...", Toast.LENGTH_SHORT).show();
	    }
	
	    private class BookNumAsyncTask extends AsyncTask<Void, Void, Integer> {
	        @Override
	        protected Integer doInBackground(Void... params) {
	            return getListNum();
	        }
	
	        @Override
	        protected void onPostExecute(Integer integer) {
	            Toast.makeText(getApplicationContext(), "图书数量: " + integer, Toast.LENGTH_SHORT).show();
	        }
	    }
	
	    private class BookListAsyncTask extends AsyncTask<Void, Void, List<Book>> {
	        @Override
	        protected List<Book> doInBackground(Void... params) {
	            List<Book> list = null;
	            try {
	                list = mRemoteBookManager.getBookList();
	            } catch (RemoteException e) {
	                e.printStackTrace();
	            }
	            return list;
	        }
	
	        @Override
	        protected void onPostExecute(List<Book> books) {
	            String content = "";
	            for (int i = 0; i < books.size(); ++i) {
	                content += books.get(i).toString() + "\n";
	            }
	            mTvBookList.setText(content);
	        }
	    }
	
	    /**
	     * 获取列表的图书数量
	     *
	     * @return 数量
	     */
	    private int getListNum() {
	        int num = 0;
	        if (mRemoteBookManager != null) {
	            try {
	                List<Book> list = mRemoteBookManager.getBookList();
	                num = list.size();
	            } catch (RemoteException e) {
	                e.printStackTrace();
	            }
	        }
	        return num;
	    }
	
	    @Override
	    protected void onDestroy() {
	        if (mRemoteBookManager != null && mRemoteBookManager.asBinder().isBinderAlive()) {
	            try {
	                Log.e(TAG, "解除注册");
	                mRemoteBookManager.unregisterListener(mOnNewBookArrivedListener);
	            } catch (RemoteException e) {
	                e.printStackTrace();
	            }
	        }
	        try {
	            unbindService(mConnection);
	        } catch (Exception e) {
	            e.printStackTrace();
	        }
	        super.onDestroy();
	    }
	
	
	}

## AIDL 客户端(另一个app)

	首先你要将aidl的文件以及bean类的文件，按包的名字照搬过来，然后再编译下。

	-MainActivity.java

	package com.vphealthy.testaidl;
	
	import androidx.appcompat.app.AppCompatActivity;
	
	import android.content.ComponentName;
	import android.content.Intent;
	import android.content.ServiceConnection;
	import android.os.Bundle;
	import android.os.IBinder;
	import android.os.RemoteException;
	import android.util.Log;
	import android.view.View;
	import android.widget.Button;
	import android.widget.TextView;
	
	import com.timaimee.demohband.aidldemo.Book;
	import com.timaimee.demohband.aidldemo.IBookManager;
	import com.timaimee.demohband.aidldemo.IOnNewBookArrivedListener;
	
	import java.util.List;
	
	public class MainActivity extends AppCompatActivity implements View.OnClickListener {
	    String TAG = "MainActivity";
	    IBookManager mBookManger;
	    Button mBinder, mAddBook;
	    TextView mTextView;
	    IOnNewBookArrivedListener.Stub listener = new IOnNewBookArrivedListener.Stub() {
	        @Override
	        public void onNewBookArrived(Book newBook) throws RemoteException {
	            Log.i(TAG, "get listenter callback");
	            getManagerBook();
	        }
	    };
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_main);
	        mTextView = findViewById(R.id.tv_book_list);
	        mBinder = findViewById(R.id.bind_aidl);
	        mAddBook = findViewById(R.id.add_book);
	        mBinder.setOnClickListener(this);
	        mAddBook.setOnClickListener(this);
	    }
	
	    @Override
	    public void onClick(View view) {
	        int id = view.getId();
	        switch (id) {
	            case R.id.bind_aidl:
	                binderService();
	                break;
	            case R.id.add_book:
	                addBook();
	                break;
	        }
	    }
	
	    private void addBook() {
	        Book newBook = new Book("学姐的故事", 3);
	        try {
	            mBookManger.addBook(newBook);
	        } catch (RemoteException e) {
	            e.printStackTrace();
	        }
	    }
	
	    private void binderService() {
	        Log.i(TAG, "binderService");
	        Intent intent = new Intent();
	        intent.setPackage("com.timaimee.demohband");
	        intent.setAction("lijinliang");
	        bindService(intent, new ServiceConnection() {
	            @Override
	            public void onServiceConnected(ComponentName name, IBinder service) {
	                Log.i(TAG, "onServiceConnected");
	                mBookManger = IBookManager.Stub.asInterface(service);
	                try {
	                    mBookManger.registerListener(listener);
	                } catch (RemoteException e) {
	                    Log.i(TAG, "onServiceConnected RemoteException");
	                    e.printStackTrace();
	                }
	            }
	
	            @Override
	            public void onServiceDisconnected(ComponentName name) {
	                Log.i(TAG, "onServiceDisconnected");
	                try {
	                    if (mBookManger != null) {
	                        mBookManger.unregisterListener(listener);
	                    }
	                } catch (RemoteException e) {
	                    e.printStackTrace();
	                }
	            }
	        }, BIND_AUTO_CREATE);
	    }
	
	    private void getManagerBook() {
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                try {
	                    final List<Book> bookList = mBookManger.getBookList();
	                    runOnUiThread(new Runnable() {
	                        @Override
	                        public void run() {
	                            String content = "";
	                            for (int i = 0; i < bookList.size(); ++i) {
	                                content += bookList.get(i).toString() + "\n";
	                            }
	                            mTextView.setText(content);
	                        }
	                    });
	                } catch (RemoteException e) {
	                    e.printStackTrace();
	                }
	            }
	        }).start();
	    }
	}
	
