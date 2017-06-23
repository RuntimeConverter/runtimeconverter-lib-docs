# Documentation for the RuntimeConverter library

The RuntimeConverter is a code coverter with an assosiated runtime library that enables the converter to fully implement all the important aspects of the PHP language into Java.

The RuntimeConverter is owned and operated by runtimeconverter.com, while online conversions are available from php2java.com.

## Platform Support

Although the runtime library is written in Java, it uses native libraries to access PHP functions and extensions. We are releasing native libraries for macOS and Ubuntu linux. We have native libraries for Windows, but will not be releasing them to avoid the resources needed to support them.

On Linux, the runtime library links dynamically with php, and so a separate php binary is available for Ubuntu Linux. On macOS, dynamic linking was not possible, and so only one native binary is needed for macOS.

The PHP implementation used is version 7.x. To query the exact version number, use the "phpversion" function.


## Intended Usage

The intended usage of conversions provided by php2java.com is as a first step in converting your website application to Java. We provide you with an application that should hopefully compile and run exactly as your old application. Nevertheless, you will likely experience various bugs resulting from the conversion. We will provide below some documentation of the internals of the runtime, but your main strategy for fixing bugs should be to convert the code to conventional Java, rather than developing with the runtime library. If there is a bug in the librray, you might report it, but better to migrate that section of your code off the runtime library.

## Production Usage and Support

We cannot support your personal or small business application in a production environment. Use only after extesive testing and at your own risk. We cannot be held liable for any malfuctions or security lapses. For larger companies, see our enterprise support options at runtimeconverter.com.

Use of the term "supported" in this document and elsewhere does not mean that we provide any warranty or expectation of support. You are welcome to raise any issues you have experienced in the "Issues" section of this repository, or privately, but they will be triaged and added to a backlog of issues and features not yet developed.

## Installation

1. Create a read-accessible folder named "/var/runtimeconverter"
2. Create read-write-accessible folders named "/var/runtimeconverter/sessionData" and "/var/runtimeconverter/temp".
3. Download your platform specific binaries and place them in the "/var/runtimeconverter" folder.
4. Create a project of some kind from your converted code zip results, such as with Eclipse or IntelliJ
5. Download the Java dependencies and include them in your project.
6. Download the main runtime library .JAR file and include it in your project.
7. Attempt to compile

## Run on Web

To run your project on web, just create a .WAR file and deploy it to a server such as Tomcat 8.

## Run as a Command Line Application

Usage: java -classpath {...} com.project.convertedCode.main.CommandLineInterface SCRIPT_FILEPATH SIMULATED_WORING_DIRECTORY

where SCRIPT_FILEPATH is the path of the include file to load, and SIMULATED_WORKING_DIRECTORY is the working directory as you will represent it to the runtime.

## Binary Download Links

The current library release version is 1.0

### Ubuntu Linux
libphp7.so: ---

runtimeconverter-JNI.so: ---

### macOS
runtimconverter-JNI-Mac.dylib: ---

## File Paths

The runtime uses simulated file paths for includes. The root directory is based on the relative paths that you uploaded in your project zip file. It is reccommended to provide a zip file with a full directory structure. For example, if your application was in /var/www, provide a zip with code in var/www. If you upload a zip with codefiles in the top level, your code will have a simulated path of /example.php.

## Converted Code Structure

Code conversions use the default package name of "com.project.convertedCode" as their root. Subpackages include:

classes.GlobalNamespace

functions.GlobalNamespace

includes

main

namespaces

servlets

"GlobalNamespace" is currently the only supported namespace. functions and classes contain functions and classes regardless of where they were defined. No overlap of names is allowed (as in PHP you could define a function in multiple places but only load one at runtime). "GlobalNamespace" is also pending a refactor to use proper lowercase "g".

"includes" and "servlets" have a structure based on the simulated filepaths described above. "includes" contains the php files minus the functions and classes, while servlets are there to manage resources and load the include referenced by its URI.

"main" contains "CommandLineInterface", which loads your CLI request, and "Project" which is currently a stub file containing only the base package name for the project.

"namespaces" is for static objects that refer to a namespace, in particular function implementation references. These could be user-space or php internal functions. "functions.GlobalNamespace" is not actually referenced in the converted code anywhere.

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

`public interface RuntimeClassInterface {

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
	
}`

"converterRuntimeCall" is a convenience method that maps to "converterRuntimeCall". Most of these functions provide a switch mechanism to map calls and property accessors without using relection. "Class caller" is the class of the calling object. For public methods/properties, any class, such as Object.class can be used, otherwise, access levels are checked.

## Global Variables

Includes use shared global variables only, so they have a special "scope" object for their variables. They are managed by the runtime. Use of the "global" keyword is also supported.

## Includes

Includes are loaded using Java relection by the "env.include" method. If you experience any issues with this, find the Java class that represents your include and its static property "instance" and call "include(RuntimeEnv env, RuntimeStack stack)" with your current stack.

Includes outside of the global scope (ie. from fuctions or class methods) are not supported.

## RuntimeEnv

The RuntimeEnv object manages many runtime features, particularly those related to resource management, sessions, request variables, and file uploads. A PHP context is created upon initialization, and multiple open RuntimeEnv objects per-thread is not supported. "closeServlet" or "close" is used to clean up resources and destroy the PHP context used for running functions.

Note that the pointer used for PHP context access a thread local and is not stored in RuntimeEnv, though it is initialized and destroyed there.

## Removing the Runtime Library (RuntimeEnv) from your code

Nearly every method of the converted code asks for a RuntimeEnv, but it is not necessarily ever used by it. The ZendFunction / LazyInitZendFunction objects, for example, do not use the RuntimeEnv object passed to them. It is merely a part of the interface specification, and can be safely replaced with "null" where the codepaths are known.

## ZNull

The variable ZNull is used in various places to better emulate PHP types. It is mostly equivelent to "null", though "null" represents PHP's "variable undefined" state.

## ZendFunctionPointer

You can use "com.runtimeconverter.jni.ZendFunctionPointer.getFunctionPointer(String function_name)" to get a PHP function callable. The class "com.runtimeconverter.runtime.functions.LazyInitZendFunction" is used by the converter to avoid a long startup delay, but is equivelent to a ZendFunctionPointer callable.

## Arrays

The PHP Array type is represented by the interface "com.runtimeconverter.runtime.arrays.ConverterRuntimeArray". Its concrete type is "ConverterRuntimeArrayContainer".

Use either

public ConverterRuntimeArrayContainer(int reserve_size)

public static ConverterRuntimeArrayContainer createWithZPairs(ZPair[] input)

as constructors.

The class ConverterRuntimeArrayContainer, along with the array functions found in "com.runtimeconverter.runtime.modules.standard.array.function_*" are extensively tested.

## ZPair

ZPair is a simple key/value store used by the "ConverterRuntimeArray" interface. It includes an additional "hashOrNumericKey" private field that compares with PHP's internal implementation where numeric keys have logical significance.
