# Documentation for the RuntimeConverter library

The RuntimeConverter is a code converter with an associated runtime library that enables the converter to fully implement all the important aspects of the PHP language into Java.

The RuntimeConverter is owned and operated by runtimeconverter.com, while online conversions are available from php2java.com.

## Platform Support

Although the runtime library is written in Java, it uses native libraries to access PHP functions and extensions. We are releasing native libraries for macOS and Ubuntu linux. We have native libraries for Windows, but will not be releasing them to avoid the resources needed to support them.

On Linux, the runtime library links dynamically with php, and so a separate php binary is available for Ubuntu Linux. On macOS, dynamic linking was not possible, and so only one native binary is needed for macOS.

The PHP implementation used is version 7.x. To query the exact version number, use the "phpversion" function.


## Intended Usage

The intended usage of conversions provided by php2java.com is as a first step in converting your website application to Java. We provide you with an application that should hopefully compile and run exactly as your old application. Nevertheless, you will likely experience various bugs resulting from the conversion. We will provide below some documentation of the internals of the runtime, but your main strategy for fixing bugs should be to convert the code to conventional Java, rather than developing with the runtime library. If there is a bug in the library, you might report it, but better to migrate that section of your code off the runtime library.

## Production Usage and Support

We cannot support your personal or small business application in a production environment. Use only after extensive testing and at your own risk. We cannot be held liable for any malfunctions or security lapses. For larger companies, see our enterprise support options at runtimeconverter.com.

Use of the term "supported" in this document and elsewhere does not mean that we provide any warranty or expectation of support. You are welcome to raise any issues you have experienced in the "Issues" section of this repository, or privately, but they will be triaged and added to a backlog of issues and features not yet developed.

Also, it is important to know that **code which uses the embedded PHP functions will always run slower** than the same code using PHP. This is due to the need to bridge function arguments and return values into php's native zval type before they can be used. If your site has an expected heavy load, you will want to convert at least the most heavily used portions to use native Java only.

## Installation

1. Only supported operating systems are MacOS and Ubunut 16.04
2. Install gradle
3. If on Ubuntu, run "sudo apt-get install mcrypt libxpm4 libjpeg8 libwebp5" to install needed extension dependencies.

## Run on Web

Run "gradle bootRun". Check the servlet class annotations in the "src/main/java/com/project/convertedCode/servlets" to see what paths to use. You may want to use a proxy server like apache or nginx to adjust the paths.

## Run as a Command Line Application

See the following blog entry for details: http://www.runtimeconverter.com/single-post/2017/09/26/Adding-Gradle-and-Spring-Boot


## Version History

The current library release version is 2.0.0.

**2017-06-28 (Version 1.1)**

**2017-10-29 (Version 2.0.0)**



## File Paths

The runtime uses simulated file paths for includes. The root directory is based on the relative paths that you uploaded in your project zip file. It is recommended to provide a zip file with a full directory structure. For example, if your application was in /var/www, provide a zip with code in /var/www. If you upload a zip with code files in the top level, your code will have a simulated path of /example.php.

## Converted Code Structure

Code conversions use the default package name of "com.project.convertedCode" as their root. Sub-packages include:

globalNamespace

includes

main

namespaces

servlets

Each namespace contains sub-packages: functions, classes, and namespaces. The "namespaces" sub-package contains the sub-namespaces that belong to that particular namespace, if any exist. "globalNamespace" is the top level namespace.

"includes" and "servlets" have a structure based on the simulated file paths described above. "includes" contains the php files minus the functions and classes, while servlets are there to manage resources and load the include referenced by its URI.

## Functions

Functions implement the com.runtimeconverter.runtime.functions.Callable interface

~~~~
public interface Callable {
	Object call(RuntimeEnv env, PassByReferenceArgs passByReference, Object... args);
	Object call(RuntimeEnv env, Object... args);
}
~~~~ 

The "PassByReferenceArgs" are not now used, and class FunctionBaseRegular maps this call to the other.

Functions are used as static objects that have no state. This was to make use of interfaces to avoid reflection in certain dynamic features (most not yet supported).

## Classes

Classes implement the "com.runtimeconverter.runtime.interfaces.RuntimeClassInterface".

~~~~
public interface RuntimeClassInterface {
	Object converterRuntimeCall(RuntimeEnv env, String method, Object... args);
	Object converterRuntimeCall(RuntimeEnv env, String method, Class caller, PassByReferenceArgs passByReferenceArgs, Object... args);
	void __set(Object key, Object value);
	Object __get(Object key);
	void __set(Object key, Object value, Class caller);
	Object __get(Object key, Class caller);
	boolean __isset(Object key, Class caller);
	RuntimeClassInterface phpClone();
	ConverterRuntimeArray convertToPHPArray();
	Map<String,Object> __getFieldsReadOnly();
}
~~~~ 

"converterRuntimeCall" is a convenience method that maps to "converterRuntimeCall". Most of these functions provide a switch mechanism to map calls and property accessors without using reflection. "Class caller" is the class of the calling object. For public methods/properties, any class, such as Object.class can be used, otherwise, access levels are checked.

## Global Variables

Includes use shared global variables only, so they have a special "scope" object for their variables. They are managed by the runtime. Use of the "global" keyword is also supported.

## Includes

Includes are loaded using Java reflection by the "env.include" method. If you experience any issues with this, find the Java class that represents your include and its static property "instance" and call "include(RuntimeEnv env, RuntimeStack stack)" with your current stack.

Includes outside of the global scope (ie. from functions or class methods) are not supported.

## RuntimeEnv

The RuntimeEnv object manages many runtime features, particularly those related to resource management, sessions, request variables, and file uploads. A PHP context is created upon initialization, and multiple open RuntimeEnv objects per-thread is not supported. "closeServlet" or "close" is used to clean up resources and destroy the PHP context used for running functions.

Note that the pointer used for PHP context access a thread local and is not stored in RuntimeEnv, though it is initialized and destroyed there.

## Removing the Runtime Library (RuntimeEnv) from your code

Nearly every method of the converted code asks for a RuntimeEnv, but it is not necessarily ever used by it. The ZendFunction / LazyInitZendFunction objects, for example, do not use the RuntimeEnv object passed to them. It is merely a part of the interface specification, and can be safely replaced with "null" where the code paths are known.

## ZNull

The variable ZNull is used in various places to better emulate PHP types. It is mostly equivalent to "null", though "null" represents PHP's "variable undefined" state.

## ZendFunctionPointer

You can use "com.runtimeconverter.jni.ZendFunctionPointer.getFunctionPointer(String function_name)" to get a PHP function callable. The class "com.runtimeconverter.runtime.functions.LazyInitZendFunction" is used by the converter to avoid a long startup delay, but is equivalent to a ZendFunctionPointer callable.

## Arrays

The PHP Array type is represented by the abstract class "com.runtimeconverter.runtime.arrays.ConverterRuntimeArray". Its concrete type is "ConverterRuntimeArrayContainer".

Use either of these methods as constructors:
~~~~ 

public ConverterRuntimeArrayContainer(int reserve_size)
public static ConverterRuntimeArrayContainer createWithZPairs(ZPair[] input)
~~~~ 

The class ConverterRuntimeArrayContainer, along with the array functions found in "com.runtimeconverter.runtime.modules.standard.array.function_*" are extensively tested.

ConverterRuntimeArrayContainer also implements the java.lang.Map interface which causes IntelliJ's debugger to render it as a "map", even though it is also a list. To view the underlying datastore, choose "View As" -> "Object".

## ZPair

ZPair is a simple key/value store used by the "ConverterRuntimeArray" interface. It includes an additional "hashOrNumericKey" private field that compares with PHP's internal implementation where numeric keys have logical significance.

## Missing Features and Roadmap

The most obvious missing feature as of version 1.0 is namespaces. We do not have any support yet for namespaces, but expect to add this soon. We also do not have support for pass-by-reference and variable method calls. Of those two, the pass-by-reference features would be next on the list of features after namespaces. We don't yet support also dynamic object creation from withing PHP internal functions such as PDO::FETCH_OBJECT.

## ZVal.assign() method

Many PHP variables, such as strings and arrays, are copy-on-write types. Since the converter does not know what kind of variable that you are accessing, the ZVal.assign method is called in various places such as function returns and variable copy statements. You can safely remove these calls unless the value is an array or string which you will be modifying.

## String mutability

There is currently no support for string mutability. For convenience, we are using Java strings which are immutable. It may prove impossible to fully support mutable strings and java.lang.string.

## Byte Arrays

We are using byte[] arrays as return values from PHP function calls to more accurately represent possible binary data. These objects are converted to java.lang.String if not sent to another buffer such as "echo" output. We plan to add binary input to PHP functions as an option so that PHP cryptography functions will work properly.

## ZVal Class

The runtime uses java.lang.Object as its "dynamic type". There is no other common base class for the various types that are converted from PHP. The ZVal.* static functions exist to convert between types, ex. ZVal.toString(Object a). The types supported are native types: String, Integer, Long, Float, Double, Boolean, null and runtime types: ConverterRuntimeArray. There is also a php resource pointer type that does not convert, as well as RuntimeClassInterface which is used by ZVal.setProperty.

The name "ZVal" is coincidentally similar to the name of the "zval" struct type which is PHP's internal storage type. This gives it a short name that has a little significance. It is not a type that can be instantiated, but a container for static type conversion methods.
