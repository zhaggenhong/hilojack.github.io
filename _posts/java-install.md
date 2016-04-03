# java-home
You may need to set JAVA_HOME:
  export JAVA_HOME="$(/usr/libexec/java_home)"
➜ > ~ /usr/libexec/java_home
/Library/Java/JavaVirtualMachines/jdk1.7.0_45.jdk/Contents/Home

export JRE_HOME=$JAVA_HOME/jre
export PATH=$PATH:$JRE_HOME/bin

# which javac
/System/Library/Frameworks/JavaVM.framework/Versions/A/Commands/javac
