# Java Coding & Logic Programming Questions

Classic programming problems often asked in technical interviews and online tests. Each problem includes the logic, complexity, and a clean Java solution. Where useful, alternate approaches are included.

---

## Table of Contents
1. [Number Problems](#1-number-problems)
2. [String Problems](#2-string-problems)
3. [Array Problems](#3-array-problems)
4. [Pattern Printing](#4-pattern-printing)
5. [Recursion](#5-recursion)
6. [Searching & Sorting](#6-searching--sorting)
7. [Linked List & Stack](#7-linked-list--stack)
8. [Java 8 Stream-Based Solutions](#8-java-8-stream-based-solutions)

---

## 1. Number Problems

### Q1. Check if a number is prime
A prime is divisible only by 1 and itself. Check divisors only up to sqrt(n).

```java
public static boolean isPrime(int n) {
    if (n < 2) return false;
    if (n == 2) return true;
    if (n % 2 == 0) return false;
    for (int i = 3; i * i <= n; i += 2) {
        if (n % i == 0) return false;
    }
    return true;
}
```
Time: O(sqrt(n))

### Q2. Print prime numbers from 1 to N (Sieve of Eratosthenes)
```java
public static void primesUpTo(int n) {
    boolean[] composite = new boolean[n + 1];
    for (int i = 2; i * i <= n; i++) {
        if (!composite[i]) {
            for (int j = i * i; j <= n; j += i) composite[j] = true;
        }
    }
    for (int i = 2; i <= n; i++) if (!composite[i]) System.out.print(i + " ");
}
```
Time: O(n log log n)

### Q3. Armstrong number
A number equals the sum of its digits each raised to the power of the digit count.
Examples: 153 = 1³ + 5³ + 3³, 9474 = 9⁴ + 4⁴ + 7⁴ + 4⁴.

```java
public static boolean isArmstrong(int n) {
    int original = n, digits = String.valueOf(n).length(), sum = 0;
    while (n > 0) {
        int d = n % 10;
        sum += Math.pow(d, digits);
        n /= 10;
    }
    return sum == original;
}
```

### Q4. Factorial of N
```java
// Iterative
public static long factorial(int n) {
    long f = 1;
    for (int i = 2; i <= n; i++) f *= i;
    return f;
}

// Recursive
public static long factorialRec(int n) {
    return n <= 1 ? 1 : n * factorialRec(n - 1);
}

// Big numbers (n > 20)
public static BigInteger factorialBig(int n) {
    BigInteger f = BigInteger.ONE;
    for (int i = 2; i <= n; i++) f = f.multiply(BigInteger.valueOf(i));
    return f;
}
```

### Q5. Fibonacci series
```java
// Iterative – O(n) time, O(1) space
public static void fibonacci(int n) {
    int a = 0, b = 1;
    for (int i = 0; i < n; i++) {
        System.out.print(a + " ");
        int next = a + b;
        a = b; b = next;
    }
}

// Recursive with memoization
public static long fib(int n, Map<Integer, Long> memo) {
    if (n < 2) return n;
    return memo.computeIfAbsent(n, k -> fib(k - 1, memo) + fib(k - 2, memo));
}
```

### Q6. Reverse a number
```java
public static int reverse(int n) {
    int rev = 0;
    while (n != 0) {
        rev = rev * 10 + n % 10;
        n /= 10;
    }
    return rev;
}
```

### Q7. Palindrome number
```java
public static boolean isPalindrome(int n) {
    return n == reverse(n);
}
```

### Q8. Sum of digits
```java
public static int sumOfDigits(int n) {
    int sum = 0;
    while (n > 0) { sum += n % 10; n /= 10; }
    return sum;
}
```

### Q9. Count digits
```java
public static int countDigits(int n) {
    if (n == 0) return 1;
    return (int) Math.log10(Math.abs(n)) + 1;
}
```

### Q10. Perfect number
A number equal to the sum of its proper divisors. (6 = 1+2+3, 28 = 1+2+4+7+14).
```java
public static boolean isPerfect(int n) {
    int sum = 1;
    for (int i = 2; i * i <= n; i++) {
        if (n % i == 0) {
            sum += i;
            if (i != n / i) sum += n / i;
        }
    }
    return n != 1 && sum == n;
}
```

### Q11. GCD and LCM
```java
public static int gcd(int a, int b) {
    return b == 0 ? a : gcd(b, a % b);
}
public static int lcm(int a, int b) {
    return a / gcd(a, b) * b;
}
```

### Q12. Power of two
```java
public static boolean isPowerOfTwo(int n) {
    return n > 0 && (n & (n - 1)) == 0;
}
```

### Q13. Swap two numbers without a temp variable
```java
a = a + b; b = a - b; a = a - b;
// Or using XOR
a = a ^ b; b = a ^ b; a = a ^ b;
```

### Q14. Automorphic number
A number whose square ends with the number itself (5²=25, 25²=625, 76²=5776).
```java
public static boolean isAutomorphic(int n) {
    long sq = (long) n * n;
    return String.valueOf(sq).endsWith(String.valueOf(n));
}
```

### Q15. Strong number
Sum of factorial of digits equals the number itself (145 = 1! + 4! + 5!).
```java
public static boolean isStrong(int n) {
    int original = n, sum = 0;
    while (n > 0) {
        int d = n % 10, fact = 1;
        for (int i = 2; i <= d; i++) fact *= i;
        sum += fact;
        n /= 10;
    }
    return sum == original;
}
```

---

## 2. String Problems

### Q16. Reverse a string
```java
// Approach 1: two-pointer in char array
public static String reverse(String s) {
    char[] c = s.toCharArray();
    for (int i = 0, j = c.length - 1; i < j; i++, j--) {
        char t = c[i]; c[i] = c[j]; c[j] = t;
    }
    return new String(c);
}

// Approach 2: built-in
new StringBuilder(s).reverse().toString();
```

### Q17. Check palindrome string
```java
public static boolean isPalindrome(String s) {
    int i = 0, j = s.length() - 1;
    while (i < j) {
        if (s.charAt(i++) != s.charAt(j--)) return false;
    }
    return true;
}
```

### Q18. Count vowels and consonants
```java
public static void countVC(String s) {
    int v = 0, c = 0;
    for (char ch : s.toLowerCase().toCharArray()) {
        if (Character.isLetter(ch)) {
            if ("aeiou".indexOf(ch) >= 0) v++; else c++;
        }
    }
    System.out.println("Vowels=" + v + " Consonants=" + c);
}
```

### Q19. Check if two strings are anagrams
Anagrams contain the same characters in any order ("listen", "silent").
```java
public static boolean isAnagram(String a, String b) {
    if (a.length() != b.length()) return false;
    int[] count = new int[26];
    for (int i = 0; i < a.length(); i++) {
        count[a.charAt(i) - 'a']++;
        count[b.charAt(i) - 'a']--;
    }
    for (int x : count) if (x != 0) return false;
    return true;
}
```

### Q20. First non-repeating character
```java
public static char firstNonRepeating(String s) {
    int[] freq = new int[256];
    for (char c : s.toCharArray()) freq[c]++;
    for (char c : s.toCharArray()) if (freq[c] == 1) return c;
    return '_';
}
```

### Q21. Count occurrences of each character
```java
Map<Character, Long> freq = s.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

### Q22. Remove duplicates from a string (keep first occurrence)
```java
public static String removeDuplicates(String s) {
    StringBuilder sb = new StringBuilder();
    Set<Character> seen = new HashSet<>();
    for (char c : s.toCharArray()) {
        if (seen.add(c)) sb.append(c);
    }
    return sb.toString();
}
```

### Q23. Reverse words in a sentence
```java
public static String reverseWords(String s) {
    String[] words = s.trim().split("\\s+");
    Collections.reverse(Arrays.asList(words));
    return String.join(" ", words);
}
```

### Q24. Check if string contains only digits
```java
public static boolean isNumeric(String s) {
    return s != null && s.matches("\\d+");
}
```

### Q25. Longest substring without repeating characters (sliding window)
```java
public static int longestUnique(String s) {
    Map<Character, Integer> last = new HashMap<>();
    int max = 0, start = 0;
    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (last.containsKey(c) && last.get(c) >= start) {
            start = last.get(c) + 1;
        }
        last.put(c, i);
        max = Math.max(max, i - start + 1);
    }
    return max;
}
```
Time: O(n)

### Q26. String compression (aabbbcc -> a2b3c2)
```java
public static String compress(String s) {
    StringBuilder sb = new StringBuilder();
    int i = 0;
    while (i < s.length()) {
        int j = i;
        while (j < s.length() && s.charAt(j) == s.charAt(i)) j++;
        sb.append(s.charAt(i)).append(j - i);
        i = j;
    }
    return sb.toString();
}
```

### Q27. Check if two strings are rotations
```java
public static boolean isRotation(String a, String b) {
    return a.length() == b.length() && (a + a).contains(b);
}
```

### Q28. Find all permutations of a string
```java
public static void permute(String s, String prefix) {
    if (s.isEmpty()) { System.out.println(prefix); return; }
    for (int i = 0; i < s.length(); i++) {
        permute(s.substring(0, i) + s.substring(i + 1), prefix + s.charAt(i));
    }
}
```

---

## 3. Array Problems

### Q29. Find min and max in an array (single pass)
```java
public static int[] minMax(int[] a) {
    int min = a[0], max = a[0];
    for (int x : a) {
        if (x < min) min = x;
        if (x > max) max = x;
    }
    return new int[]{min, max};
}
```

### Q30. Find second largest element
```java
public static int secondLargest(int[] a) {
    int first = Integer.MIN_VALUE, second = Integer.MIN_VALUE;
    for (int x : a) {
        if (x > first) { second = first; first = x; }
        else if (x > second && x != first) second = x;
    }
    return second;
}
```

### Q31. Reverse an array in-place
```java
public static void reverse(int[] a) {
    for (int i = 0, j = a.length - 1; i < j; i++, j--) {
        int t = a[i]; a[i] = a[j]; a[j] = t;
    }
}
```

### Q32. Remove duplicates from an array
```java
// Preserves order, returns new array
public static int[] removeDuplicates(int[] a) {
    return Arrays.stream(a).distinct().toArray();
}
```

### Q33. Find the missing number from 1..N
```java
public static int missing(int[] a, int n) {
    int expected = n * (n + 1) / 2;
    int actual = Arrays.stream(a).sum();
    return expected - actual;
}
```

### Q34. Find duplicate in array of 1..N
```java
public static int duplicate(int[] a) {
    Set<Integer> seen = new HashSet<>();
    for (int x : a) if (!seen.add(x)) return x;
    return -1;
}
```

### Q35. Two sum (return indices that add up to target)
```java
public static int[] twoSum(int[] a, int target) {
    Map<Integer, Integer> seen = new HashMap<>();
    for (int i = 0; i < a.length; i++) {
        int need = target - a[i];
        if (seen.containsKey(need)) return new int[]{seen.get(need), i};
        seen.put(a[i], i);
    }
    return new int[]{-1, -1};
}
```
Time: O(n)

### Q36. Kadane's algorithm – maximum subarray sum
```java
public static int maxSubArray(int[] a) {
    int best = a[0], current = a[0];
    for (int i = 1; i < a.length; i++) {
        current = Math.max(a[i], current + a[i]);
        best = Math.max(best, current);
    }
    return best;
}
```

### Q37. Move all zeros to the end (preserve order)
```java
public static void moveZeros(int[] a) {
    int idx = 0;
    for (int x : a) if (x != 0) a[idx++] = x;
    while (idx < a.length) a[idx++] = 0;
}
```

### Q38. Rotate array by K positions (right rotation)
```java
public static void rotate(int[] a, int k) {
    int n = a.length;
    k %= n;
    reverse(a, 0, n - 1);
    reverse(a, 0, k - 1);
    reverse(a, k, n - 1);
}
private static void reverse(int[] a, int i, int j) {
    while (i < j) { int t = a[i]; a[i++] = a[j]; a[j--] = t; }
}
```

### Q39. Merge two sorted arrays
```java
public static int[] merge(int[] a, int[] b) {
    int[] r = new int[a.length + b.length];
    int i = 0, j = 0, k = 0;
    while (i < a.length && j < b.length) r[k++] = a[i] <= b[j] ? a[i++] : b[j++];
    while (i < a.length) r[k++] = a[i++];
    while (j < b.length) r[k++] = b[j++];
    return r;
}
```

### Q40. Frequency of each element
```java
Map<Integer, Long> freq = Arrays.stream(a).boxed()
    .collect(Collectors.groupingBy(Function.identity(), Collectors.counting()));
```

---

## 4. Pattern Printing

### Q41. Right-angled triangle
```
*
* *
* * *
* * * *
```
```java
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) System.out.print("* ");
    System.out.println();
}
```

### Q42. Pyramid
```
   *
  * *
 * * *
* * * *
```
```java
for (int i = 1; i <= n; i++) {
    for (int j = 0; j < n - i; j++) System.out.print(" ");
    for (int j = 0; j < i; j++) System.out.print("* ");
    System.out.println();
}
```

### Q43. Number pyramid (Floyd's triangle)
```
1
2 3
4 5 6
7 8 9 10
```
```java
int num = 1;
for (int i = 1; i <= n; i++) {
    for (int j = 1; j <= i; j++) System.out.print(num++ + " ");
    System.out.println();
}
```

### Q44. Pascal's triangle
```java
for (int i = 0; i < n; i++) {
    int val = 1;
    for (int j = 0; j <= i; j++) {
        System.out.print(val + " ");
        val = val * (i - j) / (j + 1);
    }
    System.out.println();
}
```

---

## 5. Recursion

### Q45. Sum of digits using recursion
```java
public static int sumDigits(int n) {
    return n == 0 ? 0 : n % 10 + sumDigits(n / 10);
}
```

### Q46. Power: a^b
```java
// Fast exponentiation – O(log b)
public static long power(long a, long b) {
    if (b == 0) return 1;
    long half = power(a, b / 2);
    return (b % 2 == 0) ? half * half : half * half * a;
}
```

### Q47. Tower of Hanoi
```java
public static void hanoi(int n, char from, char to, char via) {
    if (n == 0) return;
    hanoi(n - 1, from, via, to);
    System.out.println("Move disk " + n + " from " + from + " to " + to);
    hanoi(n - 1, via, to, from);
}
```

---

## 6. Searching & Sorting

### Q48. Binary search (iterative)
```java
public static int binarySearch(int[] a, int target) {
    int lo = 0, hi = a.length - 1;
    while (lo <= hi) {
        int mid = lo + (hi - lo) / 2;
        if (a[mid] == target) return mid;
        if (a[mid] < target) lo = mid + 1; else hi = mid - 1;
    }
    return -1;
}
```
Time: O(log n)

### Q49. Bubble sort
```java
public static void bubbleSort(int[] a) {
    int n = a.length;
    for (int i = 0; i < n - 1; i++) {
        boolean swapped = false;
        for (int j = 0; j < n - 1 - i; j++) {
            if (a[j] > a[j + 1]) {
                int t = a[j]; a[j] = a[j + 1]; a[j + 1] = t;
                swapped = true;
            }
        }
        if (!swapped) break;
    }
}
```

### Q50. Quick sort
```java
public static void quickSort(int[] a, int lo, int hi) {
    if (lo < hi) {
        int p = partition(a, lo, hi);
        quickSort(a, lo, p - 1);
        quickSort(a, p + 1, hi);
    }
}
private static int partition(int[] a, int lo, int hi) {
    int pivot = a[hi], i = lo - 1;
    for (int j = lo; j < hi; j++) {
        if (a[j] <= pivot) {
            i++;
            int t = a[i]; a[i] = a[j]; a[j] = t;
        }
    }
    int t = a[i + 1]; a[i + 1] = a[hi]; a[hi] = t;
    return i + 1;
}
```

### Q51. Merge sort
```java
public static void mergeSort(int[] a, int l, int r) {
    if (l >= r) return;
    int mid = (l + r) / 2;
    mergeSort(a, l, mid);
    mergeSort(a, mid + 1, r);
    merge(a, l, mid, r);
}
private static void merge(int[] a, int l, int mid, int r) {
    int[] tmp = new int[r - l + 1];
    int i = l, j = mid + 1, k = 0;
    while (i <= mid && j <= r) tmp[k++] = a[i] <= a[j] ? a[i++] : a[j++];
    while (i <= mid) tmp[k++] = a[i++];
    while (j <= r) tmp[k++] = a[j++];
    System.arraycopy(tmp, 0, a, l, tmp.length);
}
```

---

## 7. Linked List & Stack

### Q52. Reverse a singly linked list
```java
public static Node reverse(Node head) {
    Node prev = null, curr = head;
    while (curr != null) {
        Node next = curr.next;
        curr.next = prev;
        prev = curr;
        curr = next;
    }
    return prev;
}
```

### Q53. Detect cycle in a linked list (Floyd's algorithm)
```java
public static boolean hasCycle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
        if (slow == fast) return true;
    }
    return false;
}
```

### Q54. Find middle of linked list
```java
public static Node middle(Node head) {
    Node slow = head, fast = head;
    while (fast != null && fast.next != null) {
        slow = slow.next;
        fast = fast.next.next;
    }
    return slow;
}
```

### Q55. Balanced parentheses using stack
```java
public static boolean isBalanced(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> pair = Map.of(')', '(', ']', '[', '}', '{');
    for (char c : s.toCharArray()) {
        if (pair.containsValue(c)) stack.push(c);
        else if (pair.containsKey(c)) {
            if (stack.isEmpty() || stack.pop() != pair.get(c)) return false;
        }
    }
    return stack.isEmpty();
}
```

---

## 8. Java 8 Stream-Based Solutions

### Q56. Sum of even numbers in a list
```java
int sum = list.stream().filter(n -> n % 2 == 0).mapToInt(Integer::intValue).sum();
```

### Q57. Count strings starting with a given letter
```java
long count = list.stream().filter(s -> s.startsWith("A")).count();
```

### Q58. Group employees by department
```java
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

### Q59. Find max-salary employee per department
```java
Map<String, Optional<Employee>> topPerDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))));
```

### Q60. Sort list of objects by multiple fields
```java
list.sort(Comparator.comparing(Employee::getDepartment)
                    .thenComparing(Employee::getSalary, Comparator.reverseOrder()));
```

### Q61. Convert list of strings to comma-separated string
```java
String joined = list.stream().collect(Collectors.joining(", "));
```

### Q62. Find the nth highest salary
```java
Optional<Double> nth = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(n - 1)
    .findFirst();
```

### Q63. Convert list to map
```java
Map<Long, String> idToName = employees.stream()
    .collect(Collectors.toMap(Employee::getId, Employee::getName));
```

### Q64. Partition numbers into even/odd
```java
Map<Boolean, List<Integer>> parts = list.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
```

### Q65. Find duplicates in a list
```java
Set<String> seen = new HashSet<>();
Set<String> duplicates = list.stream()
    .filter(s -> !seen.add(s))
    .collect(Collectors.toSet());
```
