# Documentation for the RuntimeConverter library

The RuntimeConverter is a code coverter with an assosiated runtime library that enables the converter to fully implement all the important aspects of the PHP language into Java.

The RuntimeConverter is owned and operated by runtimeconverter.com, while online conversions are available from php2java.com.

## Platform Support

Although the runtime library is written in Java, it uses native libraries to access PHP functions and extensions. We are releasing native libraries for macOS and Ubuntu linux. We have native libraries for Windows, but will not be releasing them to avoid the resources needed to support them.

On Linux, the runtime library links dynamically with php, and so a separate php binary is available for Ubuntu Linux. On macOS, dynamic linking was not possible, and so only one native binary is needed for macOS.


## Intended Usage

The intended usage of conversions provided by php2java.com is as a first step in converting your website application to Java. We provide you with an application that should hopefully compile and run exactly as your old application. Nevertheless, you will likely experience various bugs resulting from the conversion. We will provide below some documentation of the internals of the runtime, but your main strategy for fixing bugs should be to convert the code to conventional Java, rather than developing with the runtime library. If there is a bug in the librray, you might report it, but better to migrate that section of your code off the runtime library.

## Production Usage and Support

Unless you have an enterpise support contract, using the runtime in a production environment is NOT currently supported or intended as a use of your conversion. If you choose to do so, we are not liable for any issues you experience.

There is no reason why it could not be used in a production system given enough testing, but we cannot support your personal or small business application. For larger companies, see our enterprise support options at runtimeconverter.com.

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

"GlobalNamespace" is currently the only supported namespace. functions and classes contain functions and classes regardless of where they were defined. No overlap of names is allowed (as in PHP you could define a function in multiple places but only load one at runtime).

"includes" and "servlets" have a structure based on the simulated filepaths described above. "includes" contains the php files minus the functions and classes, while servlets are there to manage resources and load the include referenced by its URI.

"main" contains "CommandLineInterface", which loads your CLI request, and "Project" which is currently a stub file containing only the base package name for the project.

"namespaces" is for static objects that refer to a namespace, in particular function implementation references. These could be user-space or php internal functions.

