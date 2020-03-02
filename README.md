# JVMs Compare
## JVM comparisons - using benchmarks

This repository hosts a few unit tests run as a maven project with different profiles.

The unit tests rely on [**Eclipse Collections**](https://eclipse.org/collections) library and 
use JMH for benchmarks.

Various profiles are intended to execute the same benchmarks on different JVMs, such as 
GraalVM CE, GraalVM EE, OpenJDK 11, Oracle JDK 11.

## What is currently tested?

Sequence | JDK | Version |  Notes
-------------- | ------------------ | ---------------------- | -------------------------------
01 | Oracle JDK 11 | 11.0.6 | 
02 | Graal VM EE | 19.3.1 | 
03 | Graal VM CE | 19.3.1 | 
04 | AdoptOpenJDK 11 w/Hotspot | 11.0.6 +10 | 
05 | AdoptOpenJDK 11 w/OpenJ9 | 11.0.6 +10 | OpenJ9 build openj9-0.18.1
06 | OpenJDK 11 | 11.0.2 | 
07 | OpenJDK 11 w/Graal | 11.0.2 | Experimental Option +UseJVMCICompiler
08 | GraalEE w/C2 | 19.3.1 | Experimental Option -UseJVMCICompiler

## Links to download the JDKs (Mac versions used, linked)
1. Oracle JDK 11.0.6: https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html  
**Direct Link**: N/A - Requires OTN login  
_This JDK is renamed to oraclejdk-11.0.6.jdk/_  

1. GraalVM CE 19.3.1: https://github.com/graalvm/graalvm-ce-builds/releases  
**Direct Link**: https://github.com/graalvm/graalvm-ce-builds/releases/download/vm-19.3.1/graalvm-ce-java11-darwin-amd64-19.3.1.tar.gz  

1. GraalVM EE 19.3.1: https://www.oracle.com/downloads/graalvm-downloads.html  
**Direct Link**: N/A - Requires OTN login  

1. AdoptOpenJDK Hotspot 11.0.6 +10: https://adoptopenjdk.net/?variant=openjdk11&jvmVariant=hotspot  
**Direct Link**: https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.6%2B10/OpenJDK11U-jdk_x64_mac_hotspot_11.0.6_10.pkg  

1. AdoptOpenJDK OpenJ9 11.0.6 +10: https://adoptopenjdk.net/?variant=openjdk11&jvmVariant=openj9  
**Direct Link**: https://github.com/AdoptOpenJDK/openjdk11-binaries/releases/download/jdk-11.0.6%2B10_openj9-0.18.1/OpenJDK11U-jdk_x64_mac_openj9_11.0.6_10_openj9-0.18.1.pkg  
 
1. OpenJDK 11.0.2: https://jdk.java.net/archive/  
**Direct Link**: https://download.java.net/java/GA/jdk11/9/GPL/openjdk-11.0.2_osx-x64_bin.tar.gz  


#### NOTE: OS X Catalina Security - trusted executables.
It is important to test each downloaded JDK before running the benchmarks, since some of the distrubutors 
are not yet endorsed. Run the test_jdks.sh.

If any JDK throws a security constraint, the quick work-around is to remove the quarantine attribute:
For instance for GraalVM EE 19.3.1:

`sudo xattr -d com.apple.quarantine /Library/Java/JavaVirtualMachines/graalvm-ee-java11-19.3.1/`

## Setup Steps

1. Setup an IDE if you wish to include more Benchmarks or edit configurations.
1. Setup the JDKs that are being tested.
1. Setup Apache Maven (v3.6.2 or above).
1. Setup the environment variables (at a minimum, GRAALVM_EE_HOME and ORACLE_JAVA8_HOME).


## Files and directories (all current scripts are Mac specific)
* ***`output/`*** : Created after a run, stores the maven execution output (and raw benchmark data therein.  
* ***`benchmark-results/`*** : Created after a run, stores the benchmark CSV files in appropriate sub-directories.  
* **`setup.sh`** : Deletes output and benchmark directories and recreates them. Not usually needed directly.  
* **`env.sh`** : Sets the JDK HOME for each JDK used in the test. Required to update if JDK installations have different names from what is standard.   
* **`test_jdks.sh`** : This script tests the env.sh setup and prints out each JDK's version information to verify that the JDK is correctly setup.
* **`toolchains.xml`** : Local toolchains for use in maven. These point to the JDK installations and specify keywords that allow for selection when executing the maven profile for each JDK.
* **`##_*.sh`** : The numbered shell scripts that execute the maven commands to compile and execute each set of benchmarks for the JDK mentioned in the file name.
* **`pom.xml`** : The maven pom. Contains properties for the benchmark volume, forks, warmups and measurements. Also containts the profiles for each JDK with maven exec executing each benchmark class. The first benchmark in most cases is the default exec, while the rest have `exec@...` associated with them.




## Usage

Different benchmark tests can be run using different profiles set up in the `pom.xml`. More profiles
can be created by:

1. copy/pasting an existing profile in the `pom.xml`.
1. renaming the profile _**appropriately**, to reflect the toolchain and type of benchmark_.
1. modifying the toolchain configuration in **maven-toolchains-plugin** to pick the right toolchain.
1. modifying the env.sh to point to the right JDK Home.
1. {optionally} updating the **exec-maven-plugin** to run the appropriate Benchmark class.

## Notes on toolchains

Until Apache Maven 3.6.2, there was a bug that prevents using an environment variable in the 
`toolchains.xml`. This means that toolchains.xml will be different on different machines. 

Update the `toolchains.xml` locally and point to:
 
* the location of the toolchain's JDK

Additional toolchains can also be added if more JDK distros are being tested.

## Run ALL Benchmarks

There is a root run-all.sh that runas all 8 JDKs against the 7 benchmark classes. 

**This shell script deletes all existing results**

You can also run individual shell scripts that a re prefixed with a number. These numbered 
shell scripts are per JDK/JVM being tested.

In order to delete existing outputs, use the below:

### Delete all maven logs

From the root folder, run:

```
find ./output/ -maxdepth 3 -type f -name "*.txt" -delete
```

### Delete all benchmark outputs in JSON

From the root folder, run:

```
find ./benchmark-results/ -maxdepth 3 -type f -name "*.json" -delete

``` 

### Delete all outputs and reset to empty

From the root folder, run:

```
./setup.sh

```


### Experimenting with Results
Notes about the results:
* The results for each benchmark run is saved in it's own respective folder.
* The benchmark results in csv as well as maven build output are saved.
* For csv files navigate to benchmark-results (For Eg. benchmark-results/int-lists-filter/)
* For maven build output navigate to output (For Eg. output/)
