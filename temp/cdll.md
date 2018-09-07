C#调用 C/C++dll的方法：

一，如果dll里面很简单，没有指针，结构体，数组的话，在C#里面直接使用dll就行了。
使用方法如下：
1，把dll放到exe的文件夹下面。

2，把dll里面的函数，在C#使用的地方，在声明一遍。如下
  C++的DLL里面的函数：
	int init_usb_interface(int argpList);

 C#里面要如下定义
	[DllImport("win_usb.dll", EntryPoint = "init_usb_interface", CallingConvention = CallingConvention.Cdecl)]
	public static extern int init_usb_interface(int win);

3，C#里面直接调用该函数，就可以了。



二，如果用C/C++dll的函数里面有指针参数的时候，需要用到ref，这个关键字。
还有参数为结构体的，还有二维数组，可以参考下面的例子。
如果还需要回调函数的话，就请查看网上资料了，就查关键字，c# c++ dll 回调函数


1，如果dll本身有头文件，就要把Dll头文件里面的定义，在C#里面定义一下。

2，#define
  C++里面 
  #define	MAX_USB_HUB_NUM			16
   
    C#里面
    const int MAX_USB_HUB_NUM = 16;

 3，enum
C++里面：
typedef enum
{
	USB_ERR_GET_HUB_HANDLER = -1,		/* Fail to get hadle to usb hubs */
}USB_IF_ERROR_CODE;

C#里面：
public enum USB_IF_ERROR_CODE
        {
            USB_ERR_GET_HUB_HANDLER = -1,       /* Fail to get hadle to usb hubs */
}

4，结构体，结构体里面包含二维数组。
C++里面：
typedef struct
{
	int num;
	char volumn[MAX_USB_STORAGE_NUM][VOLUMN_NAME_LENGTH];
}WIN_USB_STORAGE_LIST;

C#里面：
	[StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi, Pack = 1)]
        public struct WIN_USB_STORAGE_LIST
        {
            public int num;

            [MarshalAs(UnmanagedType.ByValArray, SizeConst = MAX_USB_STORAGE_NUM, ArraySubType = System.Runtime.InteropServices.UnmanagedType.ByValTStr)]
            public WV_volumn[] volumn;
        };

        [StructLayout(LayoutKind.Sequential, CharSet = CharSet.Ansi, Pack = 1)]
        public struct WV_volumn
        {
            [MarshalAs(UnmanagedType.ByValTStr, SizeConst = VOLUMN_NAME_LENGTH)]
            public string volumn;
        }

注意这儿char 使用string 来代替的，下面也有用byte类型来代替


5，函数声明调用
C++的DLL里面的函数：
	int init_usb_interface(WIN_USB_HUB_LIST *argpList);

C#里面要如下定义
	[DllImport("win_usb.dll", EntryPoint = "init_usb_interface", CallingConvention = CallingConvention.Cdecl)]
	public static extern int init_usb_interface([In, Out] ref WIN_USB_HUB_LIST win);


来一个复杂的：
C++的DLL里面的函数：
			int write_usb_device(unsigned char *argpPath,
								   unsigned int offset,
								   void *argpInBuf,
								   unsigned long argInBufSize);

C#里面的定义：
	[DllImport("win_usb.dll", EntryPoint = "write_usb_device", CallingConvention = CallingConvention.Cdecl)]
        public static extern int write_usb_device(string argpPath,
                                   int offset,
                                   ref byte argpInBuf,
                                   int argInBufSize);
这个argpInBuf，就没有用string来定义，因为这个函数的特殊用法导致用string的时候，里面的内容有一些不正确。
只能用纯字节的指针来调用。
这个时候，C#里面调用这个函数的时候
    argpInBuf 的定义如下：
   byte[] bBuffer = bBuffer = new byte[nFileLength];

  函数调用如下：
  int ret = write_usb_device(dev.dev[i].devPath, 0, ref bBuffer[0], nFileLength);
  注意bBuffer[0]，里面的0一定要加否则，会编译出错。



-------------------------
托管和非托管的概念
托管代码（Managed Code）实际上就是中间语言（IL）代码。代码编写完毕后进行编译，此时编译器把代码编译成中间语言（IL），而不是能直接在你的电脑上运行的机器码。程序集（Assembly）的文件负责封装中间语言，程序集中包含了描述所创建的方法、类以及属性的所有元数据。
托管代码在公共语言运行库（CLR）中运行。这个运行库给运行代码提供了多种服务，通常来说，公共语言运行库可以加载和验证程序集，并以此来保证中间语言的正确性。当某些方法被调用时，公共语言运行库把具体的方法编译成适合本地计算机运行的机器码，并且将编译好的机器码缓存起来，以备下次调用时使用。这个过程就是即时编译。 
注意：程序实际上是被“托管”在公共语言运行库中。随着程序集的运行，公共语言运行库会持续地提供各种服务，例如内存管理、安全管理、线程管理等等。
总结：托管代码（Managed Code）是由公共语言运行库（CLR）执行的代码，而不是由操作系统直接执行。托管代码也可以调用CLR的运行库服务和功能，比如GC、类型检查、安全支持等等。这些服务和功能提供独立与开发语言的、统一的Managed Code应用程序行为。 

非托管代码（Unmanaged Code）是指直接编译成目标计算机的机器码，这些代码只能运行在编译出这些代码的计算机上，或者是其他相同处理器或者几乎一样处理器的计算机上。
 非托管代码不能享受公共语言运行库所提供的一些服务，例如内存管理、安全管理等。 如果非托管代码需要进行内存管理等服务，就必须显式地调用操作系统的接口，通常非托管代码调用Windows SDK所提供的API来实现内存管理。 非托管程序也可以通过调用COM接口来获取操作系统服务。 注意：C#跟Visual Studio平台的其他编程语言不一样的是，C#可以创建托管程序与非托管程序。当创建的项目选择名字以MFC，ATL或者Win32开头的项目类型，那么这个项目所产生的就是非托管程序。
 总结：非托管代码（Unmanaged Code）不由CLR公共语言运行库执行，而是由操作系统直接执行的代码。


托管的，是不需要自己释放资源的，非托管的是需要自己释放资源的
所以，C/C++都是非托管的，C#是一般托管的，C#里面也有非托管的方式，一般用的比较少。

托管和非托管之间混合编程的时候，最好使用dll。
也可以用调用 COM 对象上的接口方法 
