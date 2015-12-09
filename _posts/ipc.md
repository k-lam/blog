android中的跨进程调用学习

##Binder
###aidl
####使用
http://android.blog.51cto.com/268543/537684/

这个是ApiDemo的例子，在sdk下面
find SDK -name ISecondary.aidl找找

1. 定义一个.aidl文件（接口）

	
		interface ISecondary {

		    int getPid();
		    
		    void basicTypes(int anInt, long aLong, boolean aBoolean, float 	aFloat, double aDouble, String aString);
		}	
	
	
		
2. 实现接口，写在Server端

		 private final ISecondary.Stub mSecondaryBinder = new ISecondary.Stub() {
		        public int getPid() {
		            return Process.myPid();
		        }
		        public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, String aString) {
		        }
		    };
	

3. 调用

		private ServiceConnection mSecondaryConnection = new ServiceConnection() {
		            public void onServiceConnected(ComponentName className,
		                    IBinder service) {
		                // Connecting to a secondary interface is the same as any
		                // other interface.
		                mSecondaryService = ISecondary.Stub.asInterface(service);
		                mKillButton.setEnabled(true);
		            }
		
		            public void onServiceDisconnected(ComponentName className) {
		                mSecondaryService = null;
		                mKillButton.setEnabled(false);
		            }
		        };
		        
		        bindService(new Intent(ISecondary.class.getName()),
                        mSecondaryConnection, Context.BIND_AUTO_CREATE);
	
#####解释：
首先看到定义的.aidl文件，看到生成的java类
	
	
		public interface ISecondary extends android.os.IInterface
		{
		/** Local-side IPC implementation stub class. */
		public static abstract class Stub extends android.os.Binder implements com.example.android.apis.app.ISecondary
		{
		private static final java.lang.String DESCRIPTOR = "com.example.android.apis.app.ISecondary";
		/** Construct the stub at attach it to the interface. */
		public Stub()
		{
		this.attachInterface(this, DESCRIPTOR);
		}
		/**
		 * Cast an IBinder object into an com.example.android.apis.app.ISecondary interface,
		 * generating a proxy if needed.
		 */
		public static com.example.android.apis.app.ISecondary asInterface(android.os.IBinder obj)
		{
		if ((obj==null)) {
		return null;
		}
		android.os.IInterface iin = (android.os.IInterface)obj.queryLocalInterface(DESCRIPTOR);
		if (((iin!=null)&&(iin instanceof com.example.android.apis.app.ISecondary))) {
		return ((com.example.android.apis.app.ISecondary)iin);
		}
		return new com.example.android.apis.app.ISecondary.Stub.Proxy(obj);
		}
		public android.os.IBinder asBinder()
		{
		return this;
		}
		@Override public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException
		{
		switch (code)
		{
		case INTERFACE_TRANSACTION:
		{
		reply.writeString(DESCRIPTOR);
		return true;
		}
		case TRANSACTION_getPid:
		{
		data.enforceInterface(DESCRIPTOR);
		int _result = this.getPid();
		reply.writeNoException();
		reply.writeInt(_result);
		return true;
		}
		case TRANSACTION_basicTypes:
		{
		data.enforceInterface(DESCRIPTOR);
		int _arg0;
		_arg0 = data.readInt();
		long _arg1;
		_arg1 = data.readLong();
		boolean _arg2;
		_arg2 = (0!=data.readInt());
		float _arg3;
		_arg3 = data.readFloat();
		double _arg4;
		_arg4 = data.readDouble();
		java.lang.String _arg5;
		_arg5 = data.readString();
		this.basicTypes(_arg0, _arg1, _arg2, _arg3, _arg4, _arg5);
		reply.writeNoException();
		return true;
		}
		}
		return super.onTransact(code, data, reply, flags);
		}
		private static class Proxy implements com.example.android.apis.app.ISecondary
		{
		private android.os.IBinder mRemote;
		Proxy(android.os.IBinder remote)
		{
		mRemote = remote;
		}
		public android.os.IBinder asBinder()
		{
		return mRemote;
		}
		public java.lang.String getInterfaceDescriptor()
		{
		return DESCRIPTOR;
		}
		/**
		     * Request the PID of this service, to do evil things with it.
		     */
		public int getPid() throws android.os.RemoteException
		{
		android.os.Parcel _data = android.os.Parcel.obtain();
		android.os.Parcel _reply = android.os.Parcel.obtain();
		int _result;
		try {
		_data.writeInterfaceToken(DESCRIPTOR);
		mRemote.transact(Stub.TRANSACTION_getPid, _data, _reply, 0);
		_reply.readException();
		_result = _reply.readInt();
		}
		finally {
		_reply.recycle();
		_data.recycle();
		}
		return _result;
		}
		/**
		     * This demonstrates the basic types that you can use as parameters
		     * and return values in AIDL.
		     */
		public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException
		{
		android.os.Parcel _data = android.os.Parcel.obtain();
		android.os.Parcel _reply = android.os.Parcel.obtain();
		try {
		_data.writeInterfaceToken(DESCRIPTOR);
		_data.writeInt(anInt);
		_data.writeLong(aLong);
		_data.writeInt(((aBoolean)?(1):(0)));
		_data.writeFloat(aFloat);
		_data.writeDouble(aDouble);
		_data.writeString(aString);
		mRemote.transact(Stub.TRANSACTION_basicTypes, _data, _reply, 0);
		_reply.readException();
		}
		finally {
		_reply.recycle();
		_data.recycle();
		}
		}
		}
		static final int TRANSACTION_getPid = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
		static final int TRANSACTION_basicTypes = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
		}
		/**
		     * Request the PID of this service, to do evil things with it.
		     */
		public int getPid() throws android.os.RemoteException;
		/**
		     * This demonstrates the basic types that you can use as parameters
		     * and return values in AIDL.
		     */
		public void basicTypes(int anInt, long aLong, boolean aBoolean, float aFloat, double aDouble, java.lang.String aString) throws android.os.RemoteException;
		}


注意到一个.aidl生成的java接口是实现了IInterface接口，这个接口提供一个 `IBinder asBinder()`的方法
IBinder暂时理解为android的IPC的一个协议。

同时注意到，这个接口里面有两个对象`Stub`和`Proxy`并且两个都实现了ISecondary这个接口，`Proxy`是`Stub`的内部类

#####Stub
		public static abstract class Stub extends Binder implements ISecondary

Binder是IBinder的实现类，Binder机制是android的IPC用的

![image](http://foocoder.com/images/aidl1.png)

也就是说，Stub通过继承Binder，拥有了IPC的能力。

再看看我们2. 实现，里面是直接继承Stub的，而实现是放在Server端。所以就会发现，为什么是实现时要继承Stub，是因为Stub继承了Binder，有跨进程的能力。

最后看调用，发现，我们是通过`bindService`来调用，而Intent的action是.aidl生成的接口的class名。在Service的onConnect中

	mSecondaryService = ISecondary.Stub.asInterface(service);

这样获取到ISecondary对象，asInterface方法：

	public static com.example.android.apis.app.ISecondary asInterface(android.os.IBinder obj)
		{
		if ((obj==null)) {
		return null;
		}
		android.os.IInterface iin = (android.os.IInterface)obj.queryLocalInterface(DESCRIPTOR);
		if (((iin!=null)&&(iin instanceof com.example.android.apis.app.ISecondary))) {
		return ((com.example.android.apis.app.ISecondary)iin);
		}
		return new com.example.android.apis.app.ISecondary.Stub.Proxy(obj);
		}
		
就是如果发现是本地接口，就本地返回，如果不是，就要返回Proxy，看Proxy的代码也能看到，Proxy就是做一些取数据的工作（不同进程的数据不能直接去，同一进程下，只需要通过引用就能取数据，不同进程内存地址不同（虚拟地址，地址映射））



	
