### Conditions
```java
public class ConditionalsWithBlocks {
   public static void main(String[] args) {
      int x = 5;

      if (x < 10) {
         System.out.println("I shall increment x by 10.");
         x = x + 10;
      }

      if (x < 10) {
         System.out.println("I shall increment x by 10.");
         x = x + 10;
      }

      System.out.println(x);
   }
}
```

### while
```java
int bottles = 99;
while (bottles > 0) {
    System.out.println(bottles + " bottles of beer on the wall.");
    bottles = bottles - 1;
}
```

### Doubles and Strings
```java
String a = "Achilles";
String t = "Tortoise";
double aPos = 0;
double tPos = 100;
double aSpeed = 20;
double tSpeed = 10;
double totalTime = 0;
while (aPos < tPos) { 
    System.out.println("At time: " + totalTime);
    System.out.println("    " + a + " is at position " + aPos);
    System.out.println("    " + t + " is at position " + tPos);

    double timeToReach = (tPos - aPos) / aSpeed;
    totalTime = totalTime + timeToReach;
    aPos = aPos + timeToReach * aSpeed;
    tPos = tPos + timeToReach * tSpeed;
}
```

### drawing a triangle
```java
public class Triangle {  
    public static void main(String[] args) {  
        for (int count = 1; count <= 5; count++) {  
            for (int i = 1; i <= count; i++) {  
                System.out.print("*");  
            }  
            System.out.println();  
        }  
    }  
}
```

### defining functions
```java
public static int max(int x, int y) {
    if (x > y) {
        return x;
    }
    return y;
}

public static void main(String[] args) {
    System.out.println(max(10, 15));
}
```

### array
```java
int[] numbers = new int[3];
numbers[0] = 4;
numbers[1] = 7;
numbers[2] = 10;
System.out.println(numbers[1]);

int[] numbers = new int[]{4, 7, 10};
System.out.println(numbers[1]);
System.out.println(numbers.length);
```

### exercise: max of array
```java
public class max {  
    /**  
     * Returns the maximum value from m.     */    public static int maxmax(int[] m) {  
        int ret = m[0];  
        for (int i = 1; i < m.length; i++) {  
            if (m[i] > ret) {  
                ret = m[i];  
            }  
        }  
  
        return ret;  
    }  
  
    public static void main(String[] args) {  
        int[] numbers = new int[]{9, 2, 15, 2, 22, 10, 6};  
        System.out.println(maxmax(numbers));  
    }  
}
```


### for loops
```java
public class ClassNameHere {
    /** Uses a basic for loop to sum a. */
    public static int sum(int[] a) {
      int sum = 0;
      for (int i = 0; i < a.length; i = i + 1) {
        sum = sum + a[i];
      }
      return sum;
    }
}
```


### exercise: partial sum
```java
public class BreakContinue {  
    public static void windowPosSum(int[] a, int n) {  
        /** your code here */  
        for (int i = 0; i < a.length; i++) {  
            if (a[i] < 0) {  
                continue;  
            } else {  
                int sum = 0;  
                for (int j = i; j <= i + n; j++) {  
                    if (j >= a.length) {  
                        break;  
                    } else {  
                        sum += a[j];  
                    }  
                }  
                a[i] = sum;  
            }  
        }  
    }  
  
    public static void main(String[] args) {  
        int[] a = {1, 2, -3, 4, 5, 4};  
        System.out.println(java.util.Arrays.toString(a));  
  
        int n = 3;  
        windowPosSum(a, n);  
  
        // Should print 4, 8, -3, 13, 9, 4  
        System.out.println(java.util.Arrays.toString(a));  
    }  
}
```

### enhanced for loop
- 不关心索引时使用
```java
public class EnhancedForBreakDemo {
    public static void main(String[] args) {
        String[] a = {"cat", "dog", "laser horse", "ketchup", "horse", "horbse"};

        for (String s : a) {
            for (int j = 0; j < 3; j += 1) {
                System.out.println(s);
                if (s.contains("horse")) {
                    break;
                }                
            }
        }
    }
}
```
