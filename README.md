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

There does exist an implementation of PHP for Java (http://quercus.caucho.com/), however it is under the GPL and so would require licensing to be included in a commercial project of any kind. Enterprise support clients could negotiate their own license for it.

## Conversion issues

There are many reasons a conversion might fail to produce any results at all. Known conversion issues include:

* Using elements not yet supported (ex. namespaces, yield, traits, etc)
* Submitting code with parse errors anywhere in the project
* Using functions whose definition cannot be found (this includes extensions like memcache)

If that is the case, you should check back in 6 months to a year to see if your project is now.

We hope to replace these fatal errors with output logs and to add the missing features and extensions in a later version of the converter and runtime library.

## Installation

1. Download the binary files (see links below)
2. Install Java 8 if not installed already
3. Install HomeBrew of MacPorts (macOS only) if not installed already
4. Run the installation script for you platform and verify that the finish successfully.
5. Create a project of some kind from your converted code zip results, such as with Eclipse or IntelliJ
6. Include the Java dependencies (javaLibs folder and RuntimeConverterLib.jar) and include them in your project.
7. Compile and run/deploy your project.

## Run on Web

To run your project on web, just create a .WAR file and deploy it to a server such as Tomcat 8.

## Run as a Command Line Application

Usage: java -classpath {...} com.project.convertedCode.main.CommandLineInterface SCRIPT_FILEPATH SIMULATED_WORING_DIRECTORY ARGUMENTS...

where SCRIPT_FILEPATH is the path of the include file to load, and SIMULATED_WORKING_DIRECTORY is the working directory as you will represent it to the runtime.

## Installation Scripts

We have prepared install scripts for Ubuntu 16.04 and macOS (with MacPorts or HomeBrew). You can find the script to run in the binary archive below.

For all installations, you should have a valid version of Java 8 available. On MacOS this requires the JDK. We have not tested OpenJDK, so you should use Oracle Java.

On macOS, you will need to have installed either MacPorts or HomeBrew, as well as installed the Xcode command line tools and license (as a part of MacPorts and HomeBrew installation).

If you experience any missing dependencies, you can re-run the scripts safely. At the end of installation, you should see the embedded PHP version number, and you can re-run that test with the "run_test.sh" script.

## What the install scripts do
1. Create a read-accessible folder named "/var/runtimeconverter"
2. Create read-write-accessible folders named "/var/runtimeconverter/sessionData" and "/var/runtimeconverter/temp".
3. Copy native libraries to /var/runtimeconverter
4. Install dependencies required by the embedded php library.


## Binary Download Links

The current library release version is 1.0. Download libraries using the links below:

2017-06-27 (Version 1.0):

[https://s3.amazonaws.com/runtimeconverter-releases/release/1.0/runtimeConverterLib.zip](https://s3.amazonaws.com/runtimeconverter-releases/release/1.0/runtimeConverterLib.zip)

MD5: e003dc43093905ce37ced4c8c7580726

SHA1: a64fbe1363fda2d69732adbfcb32c7c46ae63b12


## File Paths

The runtime uses simulated file paths for includes. The root directory is based on the relative paths that you uploaded in your project zip file. It is recommended to provide a zip file with a full directory structure. For example, if your application was in /var/www, provide a zip with code in /var/www. If you upload a zip with code files in the top level, your code will have a simulated path of /example.php.

## Converted Code Structure

Code conversions use the default package name of "com.project.convertedCode" as their root. Sub-packages include:

classes.GlobalNamespace

functions.GlobalNamespace

includes

main

namespaces

servlets

"GlobalNamespace" is currently the only supported namespace. functions and classes contain functions and classes regardless of where they were defined. No overlap of names is allowed (as in PHP you could define a function in multiple places but only load one at runtime). "GlobalNamespace" is also pending a refactor to use proper lowercase "g".

"includes" and "servlets" have a structure based on the simulated file paths described above. "includes" contains the php files minus the functions and classes, while servlets are there to manage resources and load the include referenced by its URI.

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
