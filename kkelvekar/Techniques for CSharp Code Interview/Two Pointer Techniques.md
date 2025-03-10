Below are several **two-pointer** techniques in C# with examples. Each illustrates how using two pointers (often called `left` and `right`) can solve common array/string problems efficiently.

---

# 1. Find Two Numbers That Sum to a Target (Sorted Array)

**Problem**: You have a **sorted** array and a target value. Determine if there are two numbers in the array that add up to the target.

**Key Idea**:

- Start `left` at the beginning (`0`) and `right` at the end (`length - 1`).
- Calculate `sum = arr[left] + arr[right]`.
    - If `sum == target`, success!
    - If `sum < target`, move `left` up (because we need a bigger sum).
    - If `sum > target`, move `right` down (because we need a smaller sum).

```csharp
public static bool TwoSumSorted(int[] arr, int target)
{
    int left = 0;
    int right = arr.Length - 1;

    while (left < right)
    {
        int sum = arr[left] + arr[right];
        if (sum == target)
        {
            return true; // Found two numbers that add up to target
        }
        else if (sum < target)
        {
            left++;  // Need a bigger sum
        }
        else
        {
            right--; // Need a smaller sum
        }
    }

    return false; // No two numbers sum to target
}

public static void DemoTwoSumSorted()
{
    int[] sortedArray = { 1, 2, 4, 7, 11, 15 };
    int target = 9;
    bool found = TwoSumSorted(sortedArray, target);
    Console.WriteLine(found); // True (2 + 7 = 9)
}
```

**Why it works**: The array is sorted, so when `sum < target`, you can safely move `left` forward to increase the sum; when `sum > target`, move `right` backward to decrease the sum.

---

# 2. Check if a String (or Array) is a Palindrome

**Problem**: Check whether a given string reads the same forward and backward.

**Key Idea**:

- Set `left = 0` and `right = length - 1`.
- Compare `str[left]` and `str[right]`.
    - If they differ, it’s not a palindrome.
    - Otherwise, move both pointers inward and keep checking.

```csharp
public static bool IsPalindrome(string s)
{
    int left = 0;
    int right = s.Length - 1;

    while (left < right)
    {
        if (s[left] != s[right])
        {
            return false; // Mismatch => not palindrome
        }
        left++;
        right--;
    }

    return true; // Passed all checks => palindrome
}

public static void DemoPalindrome()
{
    Console.WriteLine(IsPalindrome("racecar"));  // True
    Console.WriteLine(IsPalindrome("hello"));    // False
}
```

**Variations**:

- Sometimes you ignore punctuation/spaces, so you’d skip characters at either pointer until you find a valid one to compare.
- Same logic applies to arrays of chars or integers.

---

# 3. Partition an Array Around Zero (or Any Pivot)

**Problem**: Reorder an array so that all **negative** numbers are before all **positive** numbers (a simple partition). This is similar to the partition step in QuickSort.

**Key Idea**:

- Set `left = 0` and `right = length - 1`.
- Move `left` forward until you find a positive number (that needs to go on the right).
- Move `right` backward until you find a negative number (that needs to go on the left).
- Swap them and continue.

```csharp
public static void PartitionNegativePositive(int[] arr)
{
    int left = 0;
    int right = arr.Length - 1;

    while (left < right)
    {
        // Move left pointer forward if already negative
        while (left < right && arr[left] < 0) 
            left++;
        
        // Move right pointer backward if already positive
        while (left < right && arr[right] >= 0) 
            right--;

        // Swap if out of place
        if (left < right)
        {
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
        }
    }
}

public static void DemoPartition()
{
    int[] arr = { -1, 5, 3, -2, 8, -10, 0, 7 };
    PartitionNegativePositive(arr);
    Console.WriteLine(string.Join(", ", arr));
    // Example output: -1, -2, -10, 3, 8, 5, 0, 7 
    // (All negatives on the left, all non-negatives on the right)
}
```

**Why it works**: Each swap places a negative element to the left side and a positive (or zero) element to the right side, gradually partitioning the array.

---

# 4. Remove Duplicates from a Sorted Array

**Problem**: In a **sorted** array, remove duplicates **in place** and return the new length of the array with unique elements. (Common interview question.)

**Key Idea**:

- Keep one pointer (`insertPos`) for the position of the **next unique element**.
- Use another pointer (`i`) to scan the array.
- When `arr[i]` differs from `arr[insertPos - 1]`, it’s a new element => put it at `arr[insertPos]` and increment `insertPos`.

```csharp
public static int RemoveDuplicates(int[] arr)
{
    if (arr.Length == 0) return 0;

    int insertPos = 1; // Next position to place a unique element
    for (int i = 1; i < arr.Length; i++)
    {
        if (arr[i] != arr[insertPos - 1])
        {
            arr[insertPos] = arr[i];
            insertPos++;
        }
    }

    return insertPos; // New length with unique elements
}

public static void DemoRemoveDuplicates()
{
    int[] sortedArr = { 1, 1, 2, 2, 2, 3, 4, 4 };
    int newLength = RemoveDuplicates(sortedArr);

    // The first newLength elements in sortedArr are unique
    for (int i = 0; i < newLength; i++)
    {
        Console.Write(sortedArr[i] + " ");
    }
    // Output: 1 2 3 4
}
```

**Why it works**: The array is sorted, so duplicates are always adjacent. `insertPos` lags behind to overwrite duplicates as we find new unique values.

---

# 5. Reverse an Array (In-Place)

**Problem**: Reverse an entire array (or part of an array) without extra space.

**Key Idea**:

- Set `left = 0`, `right = length - 1`.
- Swap `arr[left]` and `arr[right]`, then move `left++`, `right--` until they meet.

```csharp
public static void ReverseArray(int[] arr)
{
    int left = 0;
    int right = arr.Length - 1;

    while (left < right)
    {
        // Swap
        int temp = arr[left];
        arr[left] = arr[right];
        arr[right] = temp;

        left++;
        right--;
    }
}

public static void DemoReverseArray()
{
    int[] arr = { 1, 2, 3, 4, 5 };
    ReverseArray(arr);
    Console.WriteLine(string.Join(", ", arr)); 
    // Output: 5, 4, 3, 2, 1
}
```

---

# 6. Move Zeroes to the End (Preserve Order)

**Problem**: Given an array, move all zeroes to the end while preserving the **relative order** of non-zero elements.

**Key Idea**:

- Use a pointer `nonZeroPos` to track where the **next non-zero** element should go.
- Use another pointer `i` to scan through the array.
- If `arr[i]` is non-zero, place it at `arr[nonZeroPos]`, increment `nonZeroPos`.
- After you’ve processed all elements, fill the remainder with zeroes.

```csharp
public static void MoveZeroes(int[] arr)
{
    int nonZeroPos = 0;

    // 1. Move non-zero elements to the front
    for (int i = 0; i < arr.Length; i++)
    {
        if (arr[i] != 0)
        {
            arr[nonZeroPos] = arr[i];
            nonZeroPos++;
        }
    }

    // 2. Fill the rest with zeroes
    while (nonZeroPos < arr.Length)
    {
        arr[nonZeroPos] = 0;
        nonZeroPos++;
    }
}

public static void DemoMoveZeroes()
{
    int[] arr = { 0, 1, 0, 3, 12 };
    MoveZeroes(arr);
    Console.WriteLine(string.Join(", ", arr));
    // Output: 1, 3, 12, 0, 0
}
```

**Note**: This is sometimes called a “two-pointer approach” even though it’s more of a **slow/fast pointer** style. The main idea is that you track where the next valid element should go (`nonZeroPos`) while you scan (`i`).

---

## Summary of Two-Pointer Patterns

1. **Left & Right Collapsing**:
    
    - **Sorted Two-Sum**: If sum < target => `left++`, else => `right--`.
    - **Palindrome Check**: Compare `left` and `right` elements, move inward.
    - **Partitioning**: Move `left` forward until you find a problem, `right` backward until you find a counterpart, then swap.
2. **Slow & Fast Pointers**:
    
    - **Remove Duplicates**: `insertPos` (slow) only moves when we find a new unique element (fast pointer scanning).
    - **Move Zeroes**: `nonZeroPos` (slow) only moves when we see a non-zero (fast pointer scanning).
3. **Swapping**:
    
    - **Reverse Array**: Swap `arr[left]` and `arr[right]`, move pointers inward.

### Tips to Memorize and Apply

- **Initialize**: Typically `left = 0`, `right = arr.Length - 1`.
- **Condition**: Usually `while (left < right)` or `while (left <= right)`.
- **When to increment/decrement**:
    - If you need a **larger** sum, increment `left`.
    - If you need a **smaller** sum, decrement `right`.
    - For **partition** logic, increment or decrement pointers until you find a mismatch to swap.
- **Slow/Fast** approach is a variant for in-place transformations (remove duplicates, move zeroes, etc.).

By practicing these patterns in small coding exercises, you’ll build the “muscle memory” to apply two-pointer solutions in interviews and real projects.