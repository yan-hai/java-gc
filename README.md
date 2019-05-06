# Java GC Example
Use examples to explain the Java GC process.


## GC Root
Garbage-collection roots are always reachable and will no be garbage-collected. 
There are 4 kinds of GC roots in Java:
* **Local variables**: are kept alive by the stack of a thread. 
* **Active Java threads**: are always considered as live objects. Especially important for thread local variables
* **Static variables**: are referenced by their classes. This fact makes them de facto GC roots.
* **JNI References**: are Java ojbects that the native code has created as part of a JNI call.  

### [Local Variables Example 1](src/main/java/com/nobodyhub/learn/gc/LocalVarExample1.java)
This example shows how the GC treats the not-null local variable.
```java
package com.nobodyhub.learn.gc;

public class LocalVarExample1 {
    private int _50MB = 50 * 1024 * 1024;
    private byte[] memory = new byte[_50MB];

    public static void main(String[] args) {
        LocalVarExample1 e = new LocalVarExample1();
        System.gc();
        System.out.println("GC Completed!");
    }
}
```
The output will be:
```log
[GC [PSYoungGen: 66928K->51488K(458752K)] 66928K->51488K(983040K), 0.0351942 secs] [Times: user=0.00 sys=0.03, real=0.03 secs] 
[Full GC (System) [PSYoungGen: 51488K->0K(458752K)] [PSOldGen: 0K->51382K(524288K)] 51488K->51382K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0321382 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
GC Completed!
```
In the `Full GC`, `e` was remvoed from `PSYoungGen`, resulting in decrease of `PSYoungGen`.
Instead of being garbage-collected, the object was moved to `PSOldGen` since there is almost no change in the overall memory(10496K->10398K).

### [Local Variables Example 2](src/main/java/com/nobodyhub/learn/gc/LocalVarExample2.java)
This example shows how the GC treats the null local variable.

```java
package com.nobodyhub.learn.gc;

public class LocalVarExample2 {
    private int _10MB = 50 * 1024 * 1024;
    private byte[] memory = new byte[_10MB];

    public static void main(String[] args) {
        LocalVarExample2 e = new LocalVarExample2();
        e = null;
        System.gc();
        System.out.println("GC Completed!");
    }
}
```
This will output:
```log
[GC [PSYoungGen: 66928K->256K(458752K)] 66928K->256K(983040K), 0.0006598 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 256K->0K(458752K)] [PSOldGen: 0K->182K(524288K)] 256K->182K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0041840 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
GC Completed!
```

Instead of moving to `PSOldGen`, `e` was removed directly.

### [Local Variables Example 3](src/main/java/com/nobodyhub/learn/gc/LocalVarExample3.java)
This example shows how GC treat the local variable in the method call.

```java
package com.nobodyhub.learn.gc;

public class LocalVarExample3 {
    private int _10MB = 50 * 1024 * 1024;
    private byte[] memory = new byte[_10MB];

    public static void main(String[] args) {
        method();
        System.gc();
        System.out.println("2nd GC Completed!");
    }

    public static void method() {
        LocalVarExample3 e = new LocalVarExample3();
        System.gc();
        System.out.println("1st GC Completed!");
    }
}
```
The output will be:
```log
[GC [PSYoungGen: 66928K->51488K(458752K)] 66928K->51488K(983040K), 0.0282356 secs] [Times: user=0.01 sys=0.00, real=0.03 secs] 
[Full GC (System) [PSYoungGen: 51488K->0K(458752K)] [PSOldGen: 0K->51382K(524288K)] 51488K->51382K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0314269 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
1st GC Completed!
[GC [PSYoungGen: 23593K->32K(458752K)] 74975K->51414K(983040K), 0.0018599 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 32K->0K(458752K)] [PSOldGen: 51382K->182K(524288K)] 51414K->182K(983040K) [PSPermGen: 3379K->3379K(21248K)], 0.0044902 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
2nd GC Completed!
```

In the 1st GC, `e` was moved from `PSYoungGen` to `PSOldGen`.
In the 2nd GC, `e` was not held by any object and removed from `PSOldGen` because no method was holding any more.

### [Large Object Example](src/main/java/com/nobodyhub/learn/gc/LargeObjectExample.java)
The example show how GC deal with big object.
```java
package com.nobodyhub.learn.gc;

public class LargeObjectExample {
    private int _10MB = 500 * 1024 * 1024;
    private byte[] memory = new byte[_10MB];

    public static void main(String[] args) {
        LocalVarExample1 e = new LocalVarExample1();
        System.gc();
        System.out.println("GC Completed!");
    }
}
```
This will output:
```log
[GC [PSYoungGen: 15728K->304K(458752K)] 527728K->512304K(983040K), 0.0009057 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 304K->0K(458752K)] [PSOldGen: 512000K->512182K(524288K)] 512304K->512182K(983040K) [PSPermGen: 3377K->3377K(21248K)], 0.0042538 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
GC Completed!
```
The large object will be directly created in `PSOldGen`.

### [Class Variables Example 1](src/main/java/com/nobodyhub/learn/gc/ClassVarExample1.java)
This example shows how the GC treat static variable of class.
```java
package com.nobodyhub.learn.gc;

public class StaticVarExample {
    private int _50MB = 50 * 1024 * 1024;
    private byte[] memory = new byte[_50MB];
    private static StaticVarExample instance;

    public static void main(String[] args) {
        StaticVarExample e = new StaticVarExample();
        e.instance = new StaticVarExample();
        e = null;
        System.gc();
        System.out.println("GC Completed!");
    }
}
```
This will output:
```log
[GC [PSYoungGen: 118128K->51504K(458752K)] 118128K->51504K(983040K), 0.0308586 secs] [Times: user=0.00 sys=0.02, real=0.03 secs] 
[Full GC (System) [PSYoungGen: 51504K->0K(458752K)] [PSOldGen: 0K->51382K(524288K)] 51504K->51382K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0321588 secs] [Times: user=0.00 sys=0.03, real=0.03 secs] 
GC Completed! 
```

Because the `e` object was set to null, it was garbage-collected during the minor `GC`.
However, the object held by `StaticVarExample.instance` was not garbage-collected and moved to `PSOldGen`

### [Class Variables Example 2](src/main/java/com/nobodyhub/learn/gc/ClassVarExample2.java)
This example shows how the GC treat non-static variable of class.
```java
package com.nobodyhub.learn.gc;

public class StaticVarExample {
    private int _50MB = 50 * 1024 * 1024;
    private byte[] memory = new byte[_50MB];
    private StaticVarExample instance;

    public static void main(String[] args) {
        StaticVarExample e = new StaticVarExample();
        e.instance = new StaticVarExample();
        e = null;
        System.gc();
        System.out.println("GC Completed!");
    }
}
```
This will output:
```log
[GC [PSYoungGen: 118128K->288K(458752K)] 118128K->288K(983040K), 0.0005793 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 288K->0K(458752K)] [PSOldGen: 0K->182K(524288K)] 288K->182K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0046023 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
GC Completed!
```
Both `e` and `e.instance` were garbaged-collected in the minor `GC`.

### [Block Variables Example 1](src/main/java/com/nobodyhub/learn/gc/BlockVarExample1.java)
This example shows how the GC treat variable in the block.
```java
package com.nobodyhub.learn.gc;

public class BlockVarExample1 {
    private int _50MB = 50 * 1024 * 1024;
    private byte[] memory = new byte[_50MB];

    public static void main(String[] args) {
        {
            BlockVarExample1 e = new BlockVarExample1();
            System.gc();
            System.out.println("1st GC Completed!");
        }
        System.gc();
        System.out.println("2nd GC Completed!");
    }
}
```
This will output:
```log
[GC [PSYoungGen: 66928K->51488K(458752K)] 66928K->51488K(983040K), 0.0280569 secs] [Times: user=0.02 sys=0.02, real=0.03 secs] 
[Full GC (System) [PSYoungGen: 51488K->0K(458752K)] [PSOldGen: 0K->51382K(524288K)] 51488K->51382K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0316107 secs] [Times: user=0.03 sys=0.00, real=0.03 secs] 
1st GC Completed!
[GC [PSYoungGen: 23593K->32K(458752K)] 74975K->51414K(983040K), 0.0002750 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 32K->0K(458752K)] [PSOldGen: 51382K->51382K(524288K)] 51414K->51382K(983040K) [PSPermGen: 3378K->3378K(21248K)], 0.0040275 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 
2nd GC Completed!
```
Even though `e` is not accessable from outside block, it is not garbage-colleceted.

### [Block Variables Example 2](src/main/java/com/nobodyhub/learn/gc/BlockVarExample2.java)
This example shows how the GC treat variable in the block after set null.
```java

```
This will output:
```log
[GC [PSYoungGen: 66928K->288K(458752K)] 66928K->288K(983040K), 0.0008514 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 288K->0K(458752K)] [PSOldGen: 0K->182K(524288K)] 288K->182K(983040K) [PSPermGen: 3375K->3375K(21248K)], 0.0044942 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
1st GC Completed!
[GC [PSYoungGen: 23593K->32K(458752K)] 23775K->214K(983040K), 0.0001470 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 
[Full GC (System) [PSYoungGen: 32K->0K(458752K)] [PSOldGen: 182K->182K(524288K)] 214K->182K(983040K) [PSPermGen: 3378K->3378K(21248K)], 0.0039821 secs] [Times: user=0.02 sys=0.00, real=0.00 secs] 
2nd GC Completed!
```
`e` was garbaged-collected in the minor `GC` in the 1st round.





## Format of GC Log
Take the following output as an example:
```
2019-05-06T12:05:42.735+0800: [Full GC (System) [PSYoungGen: 10512K->0K(458752K)] [PSOldGen: 0K->10396K(524288K)] 10512K->10396K(983040K) [PSPermGen: 3044K->3044K(21248K)], 0.0088350 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 
```
* **2019-05-06T12:05:42.735+0800**: timestamp at which GC ran(if we add option `-XX:+PrintGCDateStamps`).
* **Full Gc**: Type of GC. It could be either `Full GC` or `GC`.
* **[PSYoungGen: 10512K->0K(458752K)]**: After the GC ran, the young generation, space came down from 10512K to 0K. Total allocated young generation space is 458752K.
* **[PSOldGen: 10396K->10396K(524288K)]**: After the GC ran the old generation space increased from 0K to 10396K and total allocated old generation space is 524288K.
* **10512K->10396K(983040K)**: After the GC ran, overall memory came down from 10512K to 10396K. Total allocated memory space is 983040K.
* **[PSPermGen: 3044K->3044K(21248K)]**: After the GC ran, there was no change in the perm generation.
* **0.0088350 secs**: Time the GC took to complete
* **[Times: user=0.00 sys=0.00, real=0.01 secs]**: 3 types of time are reported for every single GC event. 
    * **real**: is wall clock time (time from start to finish of the call). This is all elapsed time including time slices used by other processes and time the process spends blocked (for example if it is waiting for I/O to complete).
    * **user**: is the amount of CPU time spent in `user-mode code` (outside the kernel) within the process. This is only actual CPU time used in executing the process. Other processes, and the time the process spends blocked, do not count towards this figure.
    * **sys**: is the amount of CPU time spent in the `kernel` within the process. This means executing CPU time spent in system calls within the kernel, as opposed to library code, which is still running in user-space.

> Reference: https://dzone.com/articles/understanding-garbage-collection-log


## Environment
* Java: JDK 1.6.0_45 / 
* VM Options: -Xms1024m -Xmx1024m -Xmn512m -XX:+PrintGCDetails
