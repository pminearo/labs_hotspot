# Building GraalVM Locally

## Links

* [Grall Labs OpenJDK 17](https://github.com/graalvm/labs-openjdk-17)
* [GraalVM mx](https://github.com/graalvm/mx)
* [Graal](https://github.com/oracle/graal/)

## Setup

Ensure your environment has:

#### XCode

Ensure [XCode](https://apps.apple.com/us/app/xcode/id497799835?mt=12) is the latest version from the App Store:

Open a terminal:
```
$ xcodebuild -version
# If this prints an error/warning:
$ sudo xcode-select -s /Applications/Xcode.app/Contents/Developer
# Reverify
$ xcodebuild -version
```

#### Python 3

Open a terminal:

```
$ brew install python
$ cd /Applications/Python\ [VERSION]\

# Make sure you execute this as YOUR user account and not 'root'
$ ./Install\ Certificates.command
```

#### Autoconf

Open a terminal:
```
$ brew install autoconf
```

#### JDK 17

Make sure JDK 17 is installed:

Open a browser and download:
https://download.java.net/java/GA/jdk17.0.2/dfd4a8d0985749f896bed50d7138ee7f/8/GPL/openjdk-17.0.2_macos-aarch64_bin.tar.gz

Open a terminal:
```
$ cd /Library/Java/JavaVirtualMachines/
$ sudo tar -zxvf [PATH_TO_JDK_DOWNLOAD]/openjdk-17.0.2_macos-aarch64_bin.tar.gz
$ cd
# This assumes you are using ZSH and have a '.zshrc' file that sources '.zlogin'.
# If not, then make sure the following environment variables are set, and source the files.
# If no JAVA_HOME variable exists in '.zlogin', add JAVA_HOME.
# Otherwise, replace the existing JAVA_HOME variable with the path listed below.
# NOTE: Make sure you do not loose your existing JAVA_HOME.
# Ensure JAVA_HOME is in your PATH variable.
$ vi .zlogin
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-17.0.2.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$REPO_DIR/mx:[OTHER PATHS]:$PATH
:wq
$ source .zshrc
```

## Build

Here some instructions how I’m doing things right now:
* build labs-jdk17
```
# Build Labs OpenJDK17
$ cd $REPO_DIR
$ git clone https://github.com/graalvm/labs-openjdk-17
$ cd labs-openjdk-17
$ bash configure --with-debug-level=slowdebug --with-boot-jdk=$(echo /path/to/bootjdk/from/above)
# If this command fails, try this patch: https://gist.github.com/lewurm/1795f25362554f4a72b4eb6cb8001a4e
# Edit: $REPO_DIR/labs-openjdk-17/make/autoconf/toolchain.m4 
# Change: XCODE_VERSION_OUTPUT=`"$XCODEBUILD" -version 2>&1 | $HEAD -n 1`
# To:     XCODE_VERSION_OUTPUT=`"$XCODEBUILD" -version | $HEAD -n 1`
$ make graal-builder-image JOBS=9 LOG=info
# Now you have a proper labs-jdk17 build under: build/macosx-aarch64-server-release/images/graal-builder-jdk/

# Get GraalVM mx
$ cd $REPO_DIR
$ git clone https://github.com/graalvm/mx.git

# Set up Environment
# This assumes you are using ZSH and have a '.zshrc' file that sources '.zlogin'.
# If not, then make sure the following environment variables are set, and source the files.
# If no JAVA_HOME variable exists in '.zlogin', add JAVA_HOME.
# Otherwise, replace the existing JAVA_HOME variable with the path listed below.
# NOTE: Make sure you do not loose your existing JAVA_HOME.
# Ensure JAVA_HOME is in your PATH variable.
# Set MX_PYTHON to ensure GraalVM mx finds your Python3 version.
# NOTE: If you installed 'binutils' via HomeBrew, make sure your PATH has '/usr/bin' before '/opt/homebrew/bin' and '/opt/homebrew/opt/binutils/bin'
$ vi .zlogin
export JAVA_HOME=/path/to/github/project/labs-jdk17/build/macosx-aarch64-server-release/images/graal-builder-jdk/
export PATH=$HOME/bin:$JAVA_HOME/bin:$M2_HOME/bin:$HOME/Library/Python/3.10/bin:$REPO_DIR/mx:/usr/bin:/opt/homebrew/bin:$PATH:/opt/homebrew/opt/binutils/bin
export MX_PYTHON=/usr/local/bin/python3
:wq
$ source .zshrc

# Build GraalVM
$ git clone https://github.com/oracle/graal
$ cd graal/vm
$ mx --env libgraal build
# Prints path of the build result
$ mx --env libgraal graalvm-home
# Sanity check, this should print GraalVM + version: 
$ $(mx --env libgraal graalvm-home)/bin/java -version
```
