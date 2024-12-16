# java-advanced-module3
Practical task for module 3 of Java Advanced: Backend Core Program - Monitoring and Troubleshooting the Java Application

## Quiz
#### Prepare answers to following questions:
* Which interface JDK tools use to connect to JVM locally?

The JVM tool interface (JVM TI) is a standard native API that allows for native libraries to capture events and control a Java Virtual Machine (JVM) for the Java platform. 
These native libraries are sometimes called agent libraries and are often used as a basis for the Java technology-level tool APIs, such as the Java Debugger Interface (JDI) that comes with the Java Development Kit (JDK). 
Profiler tool vendors will often need to create an agent library that uses JVM TI.

* What is difference between profiling and traceability?

Profiling is the process of measuring an application or system by running an analysis tool called a profiler. 
Profiling tools can focus on many aspects: functions call times and count, memory usage, cpu load, and resource usage. 
Tracing is a specialized use of logging to record information about a program's execution.


## OutOfMemory (OOM) error troubleshooting
#### Get OOM error
Execute and press any key:
```
    java -jar -Xmx100m heap-1.0.0-SNAPSHOT.jar
```

![OutOfMemoryError](screenshots/OOME.png)

#### Use jvisualvm to observe OOM
- Execute:

```
    java -jar -Xmx100m heap-1.0.0-SNAPSHOT.jar
```
- In jvisualvm connect to our java process
- Go to "Monitor" tab
- Press any key in our application
- Observe how heap grows

![OutOfMemoryError in VisualVM](screenshots/OOME-observed_in_VisualVM.png)

#### Get heap dump
##### Using -XX:+HeapDumpOnOutOfMemoryError option
- Execute and press any key:

```
    java -jar -Xmx100m -XX:+HeapDumpOnOutOfMemoryError heap-1.0.0-SNAPSHOT.jar
```

![heapdump in VisualVM](screenshots/OOME-heapdump_in_VisualVM.png)

#### Get heap histogram
##### Using jcmd
```
    jcmd <pid> GC.class_histogram
```
here [Histogram from jcmd](./OOME-heap-histogram-jcmd)

![OQL_query1](screenshots/OOME-heap-histogram.png)


##### Using jmap
```
    jmap -histo <pid> 
```
here [Histogram from jmap](./OOME-heap-histogram-jmap)

#### Analyze heap dump
##### Using Java Visual VM
- Open retrieved heap dump in jvisualvm
- Identify memory leak

##### OQL
Execute OQL in jvisualvm:
```
    select objs from java.lang.Object[] objs where objs.length > 100
```  
![OQL_query1](screenshots/OQL-query1-VisualVM.png) 

```
    select referrers(objs) from java.lang.Object[] objs where objs.length > 100
```
![OQL_query1](screenshots/OQL-query2-VisualVM.png)
```
    select referrers(arr) from java.util.ArrayList arr where arr.size > 100
```
![OQL_query1](screenshots/OQL-query3-VisualVM.png)




## Deadlock troubleshooting
#### Get deadlock
- Execute java application that simulates deadlock:

```
    java -jar deadlock-1.0.0-SNAPSHOT.jar
```

- Get thread dump and locate lines similar to:

```
Found one Java-level deadlock:
=============================
"Thread 2":
  waiting to lock monitor 0x000000001bf40b68 (object 0x000000076b7777c8, a java.lang.Object),
  which is held by "Thread 1"
"Thread 1":
  waiting to lock monitor 0x000000001bf43608 (object 0x000000076b7777d8, a java.lang.Object),
  which is held by "Thread 2"

Java stack information for the threads listed above:
===================================================
"Thread 2":
        at com.epam.jmp.mat.deadlock.SimulateDeadLock.method2(SimulateDeadLock.java:44)
        - waiting to lock <0x000000076b7777c8> (a java.lang.Object)
        - locked <0x000000076b7777d8> (a java.lang.Object)
        at com.epam.jmp.mat.deadlock.DeadLockMain$2.run(DeadLockMain.java:18)
"Thread 1":
        at com.epam.jmp.mat.deadlock.SimulateDeadLock.method1(SimulateDeadLock.java:24)
        - waiting to lock <0x000000076b7777d8> (a java.lang.Object)
        - locked <0x000000076b7777c8> (a java.lang.Object)
        at com.epam.jmp.mat.deadlock.DeadLockMain$1.run(DeadLockMain.java:11)

Found 1 deadlock.
```

#### Get thread dump
1} jstack
```
    jstack -l <pid>
```

![threaddump-jstack](screenshots/deadlock-thread_dump-jstack.png)


2} kill -3
```
    kill -3 <pid>
```

![Thread dump from kill-3](screenshots/deadlock-thread_dump-kill3.png)

3} jvisualvm

![Thread dump in VisualVM](screenshots/deadlock-thread_dump-jVisualVM.png.png)

4} Windows (Ctrl + Break)

5} jcmd
```
    jcmd <pid> Thread.print
```

![threaddump-jcmd](screenshots/deadlock-thread_dump-jcmd.png)

here [Thread dump from jcmd - file](./deadlock-thread_dump-jcmd.tdump)

## Remote JVM profiling
Using [JMX Technology](https://docs.oracle.com/javase/8/docs/technotes/guides/management/agent.html)

For insecure remote connection use parameters:
```
    -Dcom.sun.management.jmxremote
    -Dcom.sun.management.jmxremote.port=7890
    -Dcom.sun.management.jmxremote.authenticate=false
    -Dcom.sun.management.jmxremote.ssl=false
```
```
    java -jar -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=7890 -Dcom.sun.management.jmxremote.authenticate=false -Dcom.sun.management.jmxremote.ssl=false simple-1.0.0-SNAPSHOT.jar
```
Connect to JVM using jconsole:
```
    jconsole localhost:7890
```

![JConsole output](screenshots/JConsole.png)

## Inspect a Flight Recording
Execute JVM with two special parameters:
```
    -XX:+UnlockCommercialFeatures
    -XX:+FlightRecorder
```
```
    java -jar -Xmx100m -XX:+UnlockCommercialFeatures -XX:+FlightRecorder -XX:StartFlightRecording=dumponexit=true,filename=flight.jfr heap-1.0.0-SNAPSHOT.jar
```

![Flight Recorder](screenshots/flight_recorder.png)

Enable Flight Recording on JVM without these parameters:
```
    java -jar -Xmx100m -XX:+UnlockCommercialFeatures heap-1.0.0-SNAPSHOT.jar
    jps -lvm
    jcmd <pid> JFR.start name=heap_recording filename=flight.jfr dumponexit=true
```

![Flight Recorder02](screenshots/flight_recorder02.png)

Open Java Mission Control and connect to default HotSpot of our JVM:
```
    jmc
```

![Java Mission Control](screenshots/jmc.png)

## jinfo
Print system properties and command-line flags that were used to start the JVM.
```
    java -jar simple-1.0.0-SNAPSHOT.jar
    jps
    jinfo <pid>
```
![jinfo](screenshots/jinfo.png)



