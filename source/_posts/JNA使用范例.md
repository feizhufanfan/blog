---
title: JNA使用范例
date: 2023-02-09 11:24:29
categories:
- JAVA
- JNA
tags:
- JAVA
- C++
---

<h1 style="text-align:center;">JNA使用范例</h1>

## JAVA、C++变量映射表  
![JAVA/Native 变量类型转换](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230209134103.png)


## 1. C++端程序示例生产DLL/SO库  
导出函数公共头文件`CommonHead.h`
```C++
#ifndef COMMONHEAD_H
#define COMMONHEAD_H

#ifdef WIN32
#ifdef __cplusplus
#define DLL_EXPORT_C_DECL extern "C" __declspec(dllexport)
#define DLL_IMPORT_C_DECL extern "C" __declspec(dllimport)
#define DLL_EXPORT_DECL extern __declspec(dllexport)
#define DLL_IMPORT_DECL extern __declspec(dllimport)
#define DLL_EXPORT_CLASS_DECL __declspec(dllexport)
#define DLL_IMPORT_CLASS_DECL __declspec(dllimport)
#else
#define DLL_EXPORT_DECL __declspec(dllexport)
#define DLL_IMPORT_DECL __declspec(dllimport)
#endif
#else
#ifdef __cplusplus
#define DLL_EXPORT_C_DECL extern "C"
#define DLL_IMPORT_C_DECL extern "C"
#define DLL_EXPORT_DECL extern
#define DLL_IMPORT_DECL extern
#define DLL_EXPORT_CLASS_DECL
#define DLL_IMPORT_CLASS_DECL
#else
#define DLL_EXPORT_DECL extern
#define DLL_IMPORT_DECL extern
#endif
#endif

#endif
``` 

### SDK头文件 `SDK.h`  
```C++
#ifndef SDK_H
#define SDK_H

#include <commonHead.h>

#ifdef SDK_EXPORTS
#define SDK_CLASS DLL_EXPORT_CLASS_DECL
#define SDK_API DLL_EXPORT_DECL
#define SDK_C_API DLL_EXPORT_C_DECL
#else
#define SDK_CLASS DLL_IMPORT_CLASS_DECL
#define SDK_API DLL_IMPORT_DECL
#define SDK_C_API DLL_IMPORT_C_DECL
#endif

#ifdef JNAEXPORTS    //在导出DLL给java使用时不能使用内存对齐，不然java传递数据内存与C++里无法对应导致数据传递失败
typedef struct TestStruct {
    int intvalue;
    int* ptrint;
    char* ptrchar;
}TestStruct;
#else

#if defined(WIN32)
#pragma pack(pop)
#endif

typedef struct TestStruct {
    int intvalue;
    int* ptrint;
    char* ptrchar;
}TestStruct;

#if defined(WIN32)
#pragma pack(pop)
#endif

#endif

//导出普通函数
SDK_C_API void StringHello(const char* inbuffer);

SDK_C_API int intFormJAVA(int inValue);

SDK_C_API int intptrFormJAVA(int *inValue);

//导出类
class  Test{
public:
    explicit Test(){}
    ~Test(){}
    int GetA(){return 0;}
    char* GetB(){return "ok";}
    void GetC(TestStruct* c){}
    void SetA(int a){}
    void SetB(char* b){}
    void SetC(TestStruct*  c){}
private:
    int a;
    char *b;
    TestStruct c;
};
SDK_C_API void* Create_jna_SDK();

SDK_C_API int SDK_GetA(void* SDKptr);
SDK_C_API char* SDK_GetB(void* SDKptr);
SDK_C_API void SDK_GetC(void* SDKptr,TestStruct* c);

SDK_C_API void SDK_SetA(void* SDKptr,int a);
SDK_C_API void SD_SetB(void* SDKptr,char* b);
SDK_C_API void SDK_setC(void* SDKptr,TestStruct* c);

SDK_C_API void Destroy_jna_SDK(void* SDKptr);
#endif
```  

SDK文件 `SDK.cpp`  
```C++
#define SDK_EXPORTS
#define JNAEXPORTS
#include "SDK.h"
void* Create_jna_SDK() {
    try{
        return (void *)new Test();
    }catch(...){
        return nullptr;
    }

}
int SDK_GetA(void *SDKptr) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    return _sdk->GetA();
}

char *SDK_GetB(void *SDKptr) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    return _sdk->GetB();
}

void SDK_SetA(void *SDKptr, int a) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    _sdk->Set(a);
}

void SD_SetB(void *SDKptr, char *b) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    _sdk->Set(b);
}

void SDK_GetC(void *SDKptr, TestStruct *c) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    _sdk->GetC(c);
}

void SDK_setC(void *SDKptr, TestStruct *c) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    _sdk->Set(c);
}

void StringHello(const char *inbuffer) {
    inbuffer="hello JNA";
}

int intFormJAVA(int inValue) {
    return inValue;
}

int intptrFormJAVA(int *inValue) {
    *inValue=1024;
    return *inValue;
}

void Destroy_jna_SDK(void *SDKptr) {
    auto _sdk=reinterpret_cast<Test*>(SDKptr);
    delete _sdk;
    _sdk= nullptr;
}
```  

## 2. 使用`jnaerator-0.12-shaded.jar`生产Java代码  
```bash
java -jar "$JAVA_HOME/jnaerator-0.12-shaded.jar" -runtime JNA -mode Directory -o SDk_JNA -f -library SDK Release/*.dll SDK/SDK.h


Normalizing parsed code...
Generating libraries...
Generating TestStruct.java
Generating SDKLibrary.java
#
# SUCCESS: JNAeration completed !
# Output mode is 'Directory(Bindings sources in simple file hierarchy)
#
# => 'F:\Workspace\SDk_JNA'
#

#出现以上信息表示生产成功


#生产SDK_JNA目录如下
├─lib
│  └─win64
└─sdk

```  

## 3. 在java项目中使用DLL  

修改生产的`SDKLibrary.java`文件  
```java
//原内容：
public interface SDKLibrary extends Library {
	public static final String JNA_LIBRARY_NAME = "SDK";
	public static final NativeLibrary JNA_NATIVE_LIB = NativeLibrary.getInstance(SDKLibrary.JNA_LIBRARY_NAME);
	public static final SDKLibrary INSTANCE = (SDKLibrary)Native.loadLibrary(SDKLibrary.JNA_LIBRARY_NAME, SDKLibrary.class);
	/**
	 * \u5bfc\u51fa\u666e\u901a\u51fd\u6570<br>
	 * Original signature : <code>void StringHello(const char*)</code><br>
	 * <i>native declaration : SDK\SDK.h:37</i><br>
	 * @deprecated use the safer methods {@link #StringHello(java.lang.String)} and {@link #StringHello(com.sun.jna.Pointer)} instead
	 */
	@Deprecated 
	void StringHello(Pointer inbuffer);
	/**
	 * \u5bfc\u51fa\u666e\u901a\u51fd\u6570<br>
	 * Original signature : <code>void StringHello(const char*)</code><br>
	 * <i>native declaration : SDK\SDK.h:37</i>
	 */
	void StringHello(String inbuffer);
	/**
	 * Original signature : <code>int intFormJAVA(int)</code><br>
	 * <i>native declaration : SDK\SDK.h:39</i>
	 */
	int intFormJAVA(int inValue);
	/**
	 * Original signature : <code>int intptrFormJAVA(int*)</code><br>
	 * <i>native declaration : SDK\SDK.h:41</i><br>
	 * @deprecated use the safer methods {@link #intptrFormJAVA(java.nio.IntBuffer)} and {@link #intptrFormJAVA(com.sun.jna.ptr.IntByReference)} instead
	 */
	@Deprecated 
	int intptrFormJAVA(IntByReference inValue);
	/**
	 * Original signature : <code>int intptrFormJAVA(int*)</code><br>
	 * <i>native declaration : SDK\SDK.h:41</i>
	 */
	int intptrFormJAVA(IntBuffer inValue);
	/**
	 * Original signature : <code>void* Create_jna_SDK()</code><br>
	 * <i>native declaration : SDK\SDK.h:60</i>
	 */
	Pointer Create_jna_SDK();
	/**
	 * Original signature : <code>int SDK_GetA(void*)</code><br>
	 * <i>native declaration : SDK\SDK.h:61</i>
	 */
	int SDK_GetA(Pointer SDKptr);
	/**
	 * Original signature : <code>char* SDK_GetB(void*)</code><br>
	 * <i>native declaration : SDK\SDK.h:62</i>
	 */
	Pointer SDK_GetB(Pointer SDKptr);
	/**
	 * Original signature : <code>void SDK_GetC(void*, TestStruct*)</code><br>
	 * <i>native declaration : SDK\SDK.h:63</i>
	 */
	void SDK_GetC(Pointer SDKptr, TestStruct c);
	/**
	 * Original signature : <code>void SDK_SetA(void*, int)</code><br>
	 * <i>native declaration : SDK\SDK.h:64</i>
	 */
	void SDK_SetA(Pointer SDKptr, int a);
	/**
	 * Original signature : <code>void SD_SetB(void*, char*)</code><br>
	 * <i>native declaration : SDK\SDK.h:65</i><br>
	 * @deprecated use the safer methods {@link #SD_SetB(com.sun.jna.Pointer, java.nio.ByteBuffer)} and {@link #SD_SetB(com.sun.jna.Pointer, com.sun.jna.Pointer)} instead
	 */
	@Deprecated 
	void SD_SetB(Pointer SDKptr, Pointer b);
	/**
	 * Original signature : <code>void SD_SetB(void*, char*)</code><br>
	 * <i>native declaration : SDK\SDK.h:65</i>
	 */
	void SD_SetB(Pointer SDKptr, ByteBuffer b);
	/**
	 * Original signature : <code>void SDK_setC(void*, TestStruct*)</code><br>
	 * <i>native declaration : SDK\SDK.h:66</i>
	 */
	void SDK_setC(Pointer SDKptr, TestStruct c);
	/**
	 * Original signature : <code>void Destroy_jna_SDK(void*)</code><br>
	 * <i>native declaration : SDK\SDK.h:67</i>
	 */
	void Destroy_jna_SDK(Pointer SDKptr);
}
//修改后：
public class SDKLibrary{

	Pointer per=null;
	public SDKLibrary(){
		per=CLibrary.INSTANCE.Create_jna_SDK();
	}
	private interface CLibrary  extends Library{
		public static final String JNA_LIBRARY_NAME = "SDK";
		public static final NativeLibrary JNA_NATIVE_LIB = NativeLibrary.getInstance("DLL/"+CLibrary.JNA_LIBRARY_NAME);
		public static final CLibrary INSTANCE = (CLibrary)Native.loadLibrary("DLL/"+CLibrary.JNA_LIBRARY_NAME, CLibrary.class);
		/**
		 * \u5bfc\u51fa\u666e\u901a\u51fd\u6570<br>
		 * Original signature : <code>void StringHello(const char*)</code><br>
		 * <i>native declaration : SDK\SDK.h:37</i><br>
		 * @deprecated use the safer methods {@link #StringHello(String)} and {@link #StringHello(Pointer)} instead
		 */
		@Deprecated
		void StringHello(Pointer inbuffer);
		/**
		 * \u5bfc\u51fa\u666e\u901a\u51fd\u6570<br>
		 * Original signature : <code>void StringHello(const char*)</code><br>
		 * <i>native declaration : SDK\SDK.h:37</i>
		 */
		void StringHello(String inbuffer);
		/**
		 * Original signature : <code>int intFormJAVA(int)</code><br>
		 * <i>native declaration : SDK\SDK.h:39</i>
		 */
		int intFormJAVA(int inValue);
		/**
		 * Original signature : <code>int intptrFormJAVA(int*)</code><br>
		 * <i>native declaration : SDK\SDK.h:41</i><br>
		 * @deprecated use the safer methods {@link #intptrFormJAVA(IntBuffer)} and {@link #intptrFormJAVA(IntByReference)} instead
		 */
		@Deprecated
		int intptrFormJAVA(IntByReference inValue);
		/**
		 * Original signature : <code>int intptrFormJAVA(int*)</code><br>
		 * <i>native declaration : SDK\SDK.h:41</i>
		 */
		int intptrFormJAVA(IntBuffer inValue);
		/**
		 * Original signature : <code>void* Create_jna_SDK()</code><br>
		 * <i>native declaration : SDK\SDK.h:60</i>
		 */
		Pointer Create_jna_SDK();
		/**
		 * Original signature : <code>int SDK_GetA(void*)</code><br>
		 * <i>native declaration : SDK\SDK.h:61</i>
		 */
		int SDK_GetA(Pointer SDKptr);
		/**
		 * Original signature : <code>char* SDK_GetB(void*)</code><br>
		 * <i>native declaration : SDK\SDK.h:62</i>
		 */
		Pointer SDK_GetB(Pointer SDKptr);
		/**
		 * Original signature : <code>void SDK_GetC(void*, TestStruct*)</code><br>
		 * <i>native declaration : SDK\SDK.h:63</i>
		 */
		void SDK_GetC(Pointer SDKptr, TestStruct c);
		/**
		 * Original signature : <code>void SDK_SetA(void*, int)</code><br>
		 * <i>native declaration : SDK\SDK.h:64</i>
		 */
		void SDK_SetA(Pointer SDKptr, int a);
		/**
		 * Original signature : <code>void SD_SetB(void*, char*)</code><br>
		 * <i>native declaration : SDK\SDK.h:65</i><br>
		 * @deprecated use the safer methods {@link #SD_SetB(Pointer, ByteBuffer)} and {@link #SD_SetB(Pointer, Pointer)} instead
		 */
		@Deprecated
		void SD_SetB(Pointer SDKptr, Pointer b);
		/**
		 * Original signature : <code>void SD_SetB(void*, char*)</code><br>
		 * <i>native declaration : SDK\SDK.h:65</i>
		 */
		void SD_SetB(Pointer SDKptr, ByteBuffer b);
		/**
		 * Original signature : <code>void SDK_setC(void*, TestStruct*)</code><br>
		 * <i>native declaration : SDK\SDK.h:66</i>
		 */
		void SDK_setC(Pointer SDKptr, TestStruct c);
		/**
		 * Original signature : <code>void Destroy_jna_SDK(void*)</code><br>
		 * <i>native declaration : SDK\SDK.h:67</i>
		 */
		void Destroy_jna_SDK(Pointer SDKptr);
	}

	public void StringHello(){CLibrary.INSTANCE.StringHello("Hello C++");}

	public int GetA(){return CLibrary.INSTANCE.SDK_GetA(per);}

	public void SetA(int a){CLibrary.INSTANCE.SDK_SetA(per,a);}

	public void SetC(TestStruct C){CLibrary.INSTANCE.SDK_setC(per,C);}

	public void Destroy(){CLibrary.INSTANCE.Destroy_jna_SDK(per);}


}


```

![项目结构](https://feizhufanfan.oss-cn-hangzhou.aliyuncs.com/blog/20230209154111.png)  
```java
至此便可以调用C++的DLL库了
//项目结构
├─lib
└─src
    ├─Resources
    │  └─DLL
    └─sdk

public class Main {
    public static void main(String[] args) throws InterruptedException {
        System.out.println("Hello world!");

        SDKLibrary sdk=new SDKLibrary();

        sdk.SetA(123);

        TestStruct.ByValue sd=new TestStruct.ByValue();
        
        sdk.SetC(sd);
        
        sdk.Destroy();
    }
}

``` 





