# Lab Report 5

## Debugging code in lab report 2 using JDB

In this lab report, we will be debugging the following method from lab report 2 using JDB.

```java
// ArrayExample.java

public class ArrayExamples {
    // Returns a *new* array with all the elements of the input array 
    // in reversed order
    static int[] reversed(int[] arr) {
        int[] newArray = new int[arr.length];
        for(int i = 0; i < arr.length; i += 1) {
            arr[i] = newArray[arr.length - i - 1];
        }
        return arr;
    }
}
```

To recap, we can test that this code is buggy using the following testers.

```java
// ArrayTests.java

import static org.junit.Assert.*;
import org.junit.*;

public class ArrayTests {

  @Test
  public void testReversed() {
    int[] input1 = { };
    assertArrayEquals(new int[]{ }, ArrayExamples.reversed(input1));
  }

  @Test
  public void testReversedMultipleElem() {
    int[] input1 = { 3, 4, 5 };
    int[] output = ArrayExamples.reversed(input1);
    assertArrayEquals(new int[] { 5, 4, 3 }, output);
  }

}

```

After running the tester, we get the following output (also notice that I have added -g to the compile command which adds information for debugger):

```
fong-macbook:ucsd-cse15l-lab3 Fong$ javac -g -cp .:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar *.java
fong-macbook:ucsd-cse15l-lab3 Fong$ java -cp .:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar org.junit.runner.JUnitCore ArrayTests
JUnit version 4.13.2
.E.
Time: 0.004
There was 1 failure:
1) testReversedMultipleElem(ArrayTests)
arrays first differed at element [0]; expected:<5> but was:<0>
        at org.junit.internal.ComparisonCriteria.arrayEquals(ComparisonCriteria.java:78)
        at org.junit.internal.ComparisonCriteria.arrayEquals(ComparisonCriteria.java:28)
        at org.junit.Assert.internalArrayEquals(Assert.java:534)
        at org.junit.Assert.assertArrayEquals(Assert.java:418)
        at org.junit.Assert.assertArrayEquals(Assert.java:429)
        at ArrayTests.testReversedMultipleElem(ArrayTests.java:15)
        ... 32 trimmed
Caused by: java.lang.AssertionError: expected:<5> but was:<0>
        at org.junit.Assert.fail(Assert.java:89)
        at org.junit.Assert.failNotEquals(Assert.java:835)
        at org.junit.Assert.assertEquals(Assert.java:120)
        at org.junit.Assert.assertEquals(Assert.java:146)
        at org.junit.internal.ExactComparisonCriteria.assertElementsEqual(ExactComparisonCriteria.java:8)
        at org.junit.internal.ComparisonCriteria.arrayEquals(ComparisonCriteria.java:76)
        ... 38 more

FAILURES!!!
Tests run: 2,  Failures: 1
```

Here, we can see that the test testReversedMultipleElem has failed. Now, we will run the tester using jdb to inverstigate why this code failed. We can do so using the following command:

```
fong-macbook:ucsd-cse15l-lab3 Fong$ jdb -classpath .:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar org.junit.runner.JUnitCore ArrayTests
```

Then we can tell jdb to run our code to line 15 which is 

```java
int[] output = ArrayExamples.reversed(input1);
```

in order for us to investigate the output of the method. We can do this by typing `stop at ArrayTests:16` into jdb which will stop the code from running further than line 15 and call `run` to tell jdb to run.

```
Initializing jdb ...
> stop at ArrayTests:16
Deferring breakpoint ArrayTests:16.
It will be set after the class is loaded.
> run
run org.junit.runner.JUnitCore ArrayTests
Set uncaught java.lang.Throwable
Set deferred uncaught java.lang.Throwable
> 
VM Started: JUnit version 4.13.2
Set deferred breakpoint ArrayTests:16
.
Breakpoint hit: "thread=main", ArrayTests.testReversedMultipleElem(), line=16 bci=21
16        assertArrayEquals(new int[] { 5, 4, 3 }, output);
```

From here we can type in `locals` to get data about all the local variables.

```
main[1] locals
Method arguments:
Local variables:
input1 = instance of int[3] (id=988)
output = instance of int[3] (id=988)
```

Here, we can notice an unusual behavior where the id of input1 and output is the same, suggesting that those two variables reference the same object. This shouldn't be the case since our method is design to return a new reversed array while preserving the original array.

We can access the data of the object referenced by both the variable by typing in `print <varialbe name>` like the following:

```
main[1] print output[0]
 output[0] = 0
main[1] print output[1]
 output[1] = 0
main[1] print output[2]
 output[2] = 0
main[1] print input1[0]   
 input1[0] = 0
main[1] print input1[1] 
 input1[1] = 0
main[1] print input1[2] 
 input1[2] = 0
 ```

We can see that, while `input1` should be an array `[3, 4, 5]`, it is `[0, 0, 0]`, and `output`, which should be `[5, 4, 3]`, is also pointing to the same array `[0, 0, 0]`.

Going back to the method implementation

```java
static int[] reversed(int[] arr) {
    int[] newArray = new int[arr.length];
    for(int i = 0; i < arr.length; i += 1) {
        arr[i] = newArray[arr.length - i - 1];
    }
    return arr;
}
```

We can spot the cause of this unexpected behavior. Instead of creating new array and copying data from the input array to the new array, our method created a new array and copy data from that to the original input array and returned the modified original input array.

Knowing the problem, we can fix it into the following:

```java
static int[] reversed(int[] arr) {
    int[] newArray = new int[arr.length];
    for(int i = 0; i < arr.length; i += 1) {
        newArray[arr.length - i - 1] = arr[i];
    }
    return newArray;
}
```

Now if we run the junit test, the tests would pass.

```
fong-macbook:ucsd-cse15l-lab3 Fong$ javac -g -cp .:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar *.java
fong-macbook:ucsd-cse15l-lab3 Fong$ java -cp .:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar org.junit.runner.JUnitCore ArrayTests
JUnit version 4.13.2
..
Time: 0.003

OK (2 tests)
```

We can dig deeper using `jdb`

```
fong-macbook:ucsd-cse15l-lab3 Fong$ jdb -classpath .:lib/hamcrest-core-1.3.jar:lib/junit-4.13.2.jar org.junit.runner.JUnitCore ArrayTests
Initializing jdb ...
> stop at ArrayTests:16
Deferring breakpoint ArrayTests:16.
It will be set after the class is loaded.
> run
run org.junit.runner.JUnitCore ArrayTests
Set uncaught java.lang.Throwable
Set deferred uncaught java.lang.Throwable
> 
VM Started: JUnit version 4.13.2
Set deferred breakpoint ArrayTests:16
.
Breakpoint hit: "thread=main", ArrayTests.testReversedMultipleElem(), line=16 bci=21
16        assertArrayEquals(new int[] { 5, 4, 3 }, output);

main[1] locals
Method arguments:
Local variables:
input1 = instance of int[3] (id=988)
output = instance of int[3] (id=989)
```

Calling locals after stopping at line 16, we can see that the unusual behavior, where the id of input1 and output is the same, we saw earlier is gone.

This is an example how we could use `jdb` to effectively debug our code.