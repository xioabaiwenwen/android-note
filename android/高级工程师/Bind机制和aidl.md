# aidl的简单使用
在AS中可以直接新建一个aidl文件
```java
// IManager.aidl
package gxl.com.aidldemo;

interface IManager {
   List<String> getStrings();
   void add(in String str);
}
```
然后build project一下，这样在build/generated/source/aidl下就能看到生存的Imanager.java文件了。这个文件等下再看，先来看下怎样使用。

新建一个MyService，这个Service你如果你单开一个进程，这样就是进程间的通信，否则是一个进程间的通信。aidl都是可以实现的。
```java
public class MyService extends Service {
    List<String> strings;

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return myBinder;
    }

    @Override
    public void onCreate() {
        super.onCreate();
        //做些初始化的操作
    }

    IManager.Stub myBinder = new IManager.Stub() {
        @Override
        public List<String> getStrings() throws RemoteException {
            synchronized (strings) {
                return strings;
            }
        }

        @Override
        public void add(String str) throws RemoteException {
            synchronized (strings) {
                if (strings == null) {
                    strings = new ArrayList<>();
                }
                strings.add(str);
            }
        }
    };
}
```
在Activity中进行绑定服务
```java
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        Intent intent = new Intent(this, MyService.class);
        bindService(intent, myConnect, BIND_AUTO_CREATE);
    }

    private IManager mManager;
    ServiceConnection myConnect = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            mManager = IManager.Stub.asInterface(service);
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };
    //添加按钮
    public void add(View view) throws RemoteException {
        mManager.add("hello"+new Random().nextInt(20));
        mManager.add("world");
    }
    //获取按钮
    public void get(View view) {
        try {
            List<String> strings = mManager.getStrings();
            Toast.makeText(this, strings.toString(), Toast.LENGTH_SHORT).show();
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }
}
```
# aidl文件分析
上面既然会了简单的使用后，下面开始对生成IManger.java进行分析,具体的分析看源码：

```java
/*
 * This file is auto-generated.  DO NOT MODIFY.
 * Original file: E:\\AndroidProjects\\AidlDemo\\app\\src\\main\\aidl\\gxl\\com\\aidldemo\\IManager.aidl
 */
package gxl.com.aidldemo;

//继承至IInterface(asBinder)
public interface IManager extends android.os.IInterface {
    //Stub 继承 Binder 实现 IManager 是个抽象类
    public static abstract class Stub extends android.os.Binder implements gxl.com.aidldemo.IManager{
		//Binder的唯一标识，一般使用Binder的类名表示，	
        private static final java.lang.String DESCRIPTOR = "gxl.com.aidldemo.IManager";
            
        public Stub(){
            this.attachInterface(this, DESCRIPTOR);
        }
        //用于将服务端的binder对象转换成客户端所需要的aidl接口类型的对象，
		//这种转换时区分线程的，如果客户端和服务端处于同一进程，那么池方法返回的就是服务端Service本身
		//否栈返回的时系统分装后的Stub.proxy对象
        public static gxl.com.aidldemo.IManager asInterface(android.os.IBinder obj){
            if ((obj==null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin!=null)&&(iin instanceof gxl.com.aidldemo.IManager))) {
                return ((gxl.com.aidldemo.IManager)iin);
            }
            return new gxl.com.aidldemo.IManager.Stub.Proxy(obj);
        }
		//返回当前的binder对象
        @Override public android.os.IBinder asBinder(){
            return this;
        }
		//这个方法运行在服务端中的binder线程中，当客户端发起跨进程请求时，远程会通过系统底层封装后交由此方法来处理.
		//服务端通过code可以确定客户端所请求的目标方法是什么，
		//接着从data中取出目标方法所需的参数(如果目标方法有参数的话)，然后执行目标方法。 参考：TRANSACTION_add 
		//当目标方法执行完毕后，就像reply中写入返回值(如果目标方法中有返回值的话)。 参考：TRANSACTION_getStrings
		// 如果onTransact返回为false的话，则会请求失败，可以利用这个特性来做权限验证
        @Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException{
            java.lang.String descriptor = DESCRIPTOR;
             switch (code) {
                case INTERFACE_TRANSACTION:
                {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_getStrings:
                {
                    data.enforceInterface(descriptor);
                    java.util.List<java.lang.String> _result = this.getStrings();//执行目标方法
                    reply.writeNoException();
                    reply.writeStringList(_result);//写入返回值
                    return true;
                }
                case TRANSACTION_add:
                {
                    data.enforceInterface(descriptor);
                    java.lang.String _arg0;
                    _arg0 = data.readString();//读取参数
                    this.add(_arg0);//执行方法
                    reply.writeNoException();
                    return true;
                }
                default:
                {
                    return super.onTransact(code, data, reply, flags);
                }
             }
        }

		//代理类 是Stub的内部类
        private static class Proxy implements gxl.com.aidldemo.IManager
        {
            private android.os.IBinder mRemote;
			//要代理的Binder对象
            Proxy(android.os.IBinder remote){
                 mRemote = remote;
            }
			//获取到代理的binder对象
            @Override public android.os.IBinder asBinder()
            {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor()
            {
                 return DESCRIPTOR;
            }
			// 须知：此方法运行在客户端，因为：asInterface方法中，如果是在同一个进程中调用，不会调用到这个方法，
			//    如果不在同一个进程中则会调用这个方法，这个方法被调用，运行在客户端，Stub的transact方法才会在远程调用
			// (1)创建改方法所需要的输入性Parcel对象_data  输出型Parcel对象_reply和返回值对象	 
			// (2)把这个方法的参数写入_data中(如果有参数的话)
			// (3)调用transact方法来发送rpc请求，同时当前线程会被挂起
			// (4)服务端onTransace方法会被调用，制定rpc过程返回后，当前线程继续执行，
			// (5)从_reply中取出rpc过程的返回结果
			// (6)将_reply中的数据返回
            @Override public java.util.List<java.lang.String> getStrings() throws android.os.RemoteException
            {
                android.os.Parcel _data = android.os.Parcel.obtain(); //(1)
                android.os.Parcel _reply = android.os.Parcel.obtain();//(1)
                java.util.List<java.lang.String> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getStrings, _data, _reply, 0);//(3)
                    _reply.readException();
                    _result = _reply.createStringArrayList();//(5)
                }
                finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;//(6)
            }
			//
            @Override public void add(java.lang.String str) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();//(1)
                android.os.Parcel _reply = android.os.Parcel.obtain();//(1)
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    _data.writeString(str);//(2)
                    mRemote.transact(Stub.TRANSACTION_add, _data, _reply, 0);//(3)
                    _reply.readException();
                }
                finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }
        }
        static final int TRANSACTION_getStrings = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_add = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

	//获取Strings方法
    public java.util.List<java.lang.String> getStrings() throws android.os.RemoteException;
	//添加String方法
    public void add(java.lang.String str) throws android.os.RemoteException;
}
```
# binder分析
先来篇任主席书中的binder的工作机制图
![](https://raw.githubusercontent.com/xioabaiwenwen/upload-images/master/20190321235217.png)

**简单原理：**
Binder通信采用C/S架构，从组件视角来说，包含Client、Server、ServiceManager以及binder驱动，其中ServiceManager用于管理系统中的各种服务。架构图如下所示：
![](https://upload-images.jianshu.io/upload_images/3985563-5ff2c4816543c433.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/921)

**Client进程：** 使用服务的进程。

**Server进程：** 提供服务的进程。

**ServiceManager进程：** ServiceManager的作用是将字符形式的Binder名字转化成Client中对该Binder的引用，使得Client能够通过Binder名字获得对Server中Binder实体的引用。

**Binder驱动：** 驱动负责进程之间Binder通信的建立，Binder在进程之间的传递，Binder引用计数管理，数据包在进程之间的传递和交互等一系列底层支持。

**Binder运行机制**

图中Client/Server/ServiceManage之间的相互通信都是基于Binder机制。既然基于Binder机制通信，那么同样也是C/S架构，则图中的3大步骤都有相应的Client端与Server端。

**注册服务(addService)：**Server进程要先注册Service到ServiceManager。该过程：Server是客户端，ServiceManager是服务端。

**获取服务(getService)：**Client进程使用某个Service前，须先向ServiceManager中获取相应的Service。该过程：Client是客户端，ServiceManager是服务端。

**使用服务：**Client根据得到的Service信息建立与Service所在的Server进程通信的通路，然后就可以直接与Service交互。该过程：client是客户端，server是服务端。

图中的Client,Server,Service Manager之间交互都是虚线表示，是由于它们彼此之间不是直接交互的，而是都通过与Binder驱动进行交互的，从而实现IPC通信方式。其中Binder驱动位于内核空间，Client,Server,Service Manager位于用户空间。Binder驱动和Service Manager可以看做是Android平台的基础架构，而Client和Server是Android的应用层，开发人员只需自定义实现client、Server端，借助Android的基本平台架构便可以直接进行IPC通信。


**感觉最牛逼的两篇分析文章**

[关于Binder，作为应用开发者你需要知道的全部 ](https://mp.weixin.qq.com/s?__biz=MzAxMTI4MTkwNQ==&mid=2650826010&idx=1&sn=491e295e6a6c0fe450ad7aa91b6e97cc&chksm=80b7b184b7c03892392015840e4ebc7f2c3533ce8c1a98a5dc6d6d3dd19d53562805d76f6dcb&scene=38#wechat_redirect)

[Android Bander设计与实现 - 设计篇](https://blog.csdn.net/universus/article/details/6211589)




