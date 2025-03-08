# 2379. Minimum Recolors to Get K Consecutive Black Blocks

You are given a 0-indexed string `blocks` of length `n`, where `blocks[i]` is either 'W' or 'B', representing the color of the ith block. The characters 'W' and 'B' denote the colors white and black, respectively.

You are also given an integer `k`, which is the desired number of consecutive black blocks.

In one operation, you can recolor a white block such that it becomes a black block.

Return the minimum number of operations needed such that there is at least one occurrence of `k` consecutive black blocks.

## Example 1:

- **Input:** `blocks = "WBBWWBBWBW"`, `k = 7`
- **Output:** `3`
- **Explanation:** One way to achieve 7 consecutive black blocks is to recolor the 0th, 3rd, and 4th blocks so that `blocks = "BBBBBBBWBW"`. It can be shown that there is no way to achieve 7 consecutive black blocks in less than 3 operations. Therefore, we return 3.

## Example 2:

- **Input:** `blocks = "WBWBBBW"`, `k = 2`
- **Output:** `0`
- **Explanation:** No changes need to be made, since 2 consecutive black blocks already exist. Therefore, we return 0.

## Constraints:

- `n == blocks.length`
- `1 <= n <= 100`
- `blocks[i]` is either 'W' or 'B'.
- `1 <= k <= n`

## Solution Overview

We are given a string `blocks`, where each character represents a block that is either black ('B') or white ('W') and the ability to apply an operation to change a white block black an unlimited number of times. Our goal is to find the minimum number of recoloring operations needed to create a segment of `k` consecutive black blocks.

### Approach 1: Queue

#### Intuition

Since existing black blocks don’t require recolors, our required number of operations is determined by the number of white blocks within each segment of `k` consecutive blocks. The fewer white blocks in a segment, the fewer recolors we need. This immediately tells us that our task is to identify the segment of `k` consecutive blocks that contains the fewest white blocks.

Now that we know we must evaluate all segments of length k, the natural way to approach this is to start from the beginning of the string, count the number of white blocks in the first k characters, and then slide forward one position at a time. For each step, we discard the leftmost character from the previous segment and include the next character from the string, updating our count of white blocks accordingly. This allows us to efficiently track the number of white blocks in each segment without recalculating from scratch every time.

To manage this process efficiently, we need a data structure that allows us to maintain a fixed-size window of k elements while quickly removing the oldest element and adding a new one. A queue is well-suited for this task because it follows the First-In-First-Out (FIFO) principle: the oldest element (leftmost in our segment) is removed first when shifting to the next segment, and the newest element is added at the end.

With this logic, we start by initializing a queue with the first k elements and counting the white blocks. As we slide through the string, we remove the first element in the queue and add the next character from the string, adjusting our white block count accordingly. By the end of this process, we will have checked all possible segments of k blocks, and we simply return the minimum number of white blocks found.

### Algorithm

1. Initialize `blockQueue` as a queue to hold k consecutive elements.
2. Initialize `numWhites` to 0 to track the current number of white blocks.
3. Iterate through the first k elements of `blocks`.
    - If the current element is white, increase `numWhites` by 1.
    - Add the current element to `blockQueue`.
4. Initialize `numRecolors` to `numWhites` to represent the minimum number of recolors needed to have k consecutive black blocks.
5. Iterate through the remaining elements of `blocks`, starting at index k. For each element:
    - Remove the top element of the queue and decrease `numWhites` by 1 if the top element is white.
    - Add the current element to `blockQueue` and increase `numWhites` by 1 if the element is white.
    - Update `numRecolors` to the minimum of `numRecolors` and `numWhites`.
6. Return `numRecolors`.

```py
class Solution:
    def minimumRecolors(self, blocks: str, k: int) -> int:
        block_queue = deque()
        num_whites = 0

        # Initiate queue with first k values
        for i in range(k):
            current_char = blocks[i]
            if current_char == "W":
                num_whites += 1
            block_queue.append(current_char)

        # Set initial minimum
        num_recolors = num_whites

        for i in range(k, len(blocks)):

            # Remove front element from queue and update current number of white blocks
            if block_queue.popleft() == "W":
                num_whites -= 1

            # Add current element to queue and update current number of white blocks
            current_char = blocks[i]
            if current_char == "W":
                num_whites += 1
            block_queue.append(current_char)

            # Update minimum
            num_recolors = min(num_recolors, num_whites)

        return num_recolors
```

```java
class Solution {

    public int minimumRecolors(String blocks, int k) {
        Queue<Character> blockQueue = new LinkedList<>();
        int numWhites = 0;

        // Initiate queue with first k values
        for (int i = 0; i < k; i++) {
            char currentChar = blocks.charAt(i);
            if (currentChar == 'W') numWhites++;
            blockQueue.add(currentChar);
        }

        // Set initial minimum
        int numRecolors = numWhites;

        for (int i = k; i < blocks.length(); i++) {
            // Remove front element from queue and update current number of white blocks
            if (blockQueue.poll() == 'W') numWhites--;

            // Add current element to queue and update current number of white blocks
            char currentChar = blocks.charAt(i);
            if (currentChar == 'W') numWhites++;
            blockQueue.add(currentChar);

            // Update minimum
            numRecolors = Math.min(numRecolors, numWhites);
        }
        return numRecolors;
    }
}
```

### Complexity Analysis

Let N be the length of blocks and M be the value of k.

**Time Complexity:** O(N)

The algorithm iterates through each element of blocks exactly once, performing constant-time operations on each element. Specifically in each iteration, it checks and updates blockQueue and performs arithmetic operations. Both of these operations are O(1) on average due to the use of a queue and being independent of the input size. Therefore, the overall time complexity is linear to the number of elements in blocks, O(n).

Note: The operations on blockQueue (such as front, push, and pop) are considered O(1) on average due to the nature of queues.

**Space Complexity:** O(M)

The space complexity is determined by blockQueue.

The algorithm continues adding elements to blockQueue until it contains k elements. From there, we remove an element from blockQueue before adding a new one.

As a result, the size of blockQueue is bound by k, leading to an overall space complexity of O(M).

### Approach 2: Sliding Window

#### Intuition

In the previous approach, we used a queue to manage the elements in the blocks array, but this came at the cost of additional space allocation. For each segment of k blocks, we had to store up to k characters in the queue, resulting in linear space complexity relative to k. To avoid this overhead, we need a solution that doesn't require extra space for storing the segments.

We can achieve this by adopting a Fixed Sliding Window Approach. The idea here is to slide a window of size k across the array while maintaining two pointers, left and right, that represent the start and end of the window. By incrementing both pointers together, we can efficiently track and check each segment of size k without needing extra space.

To implement this approach, we start by initializing both left and right pointers at the beginning of the array. Then, we move the right pointer until we have exactly k elements in the window, which is the range we’re interested in. Once we’ve captured a window of size k, we check how many white blocks are in this segment.

After that, we increment both left and right by one position at each step. This moves the window to the next segment, and we again check how many white blocks are present. We repeat this process until the window has slid across the entire array.

By the end, we will have checked every possible segment of k consecutive blocks. At each step, we can track and update the minimum number of recolors needed. The beauty of this approach is that it allows us to explore all potential segments without the need for any extra space, other than a few variables to track the window and the number of recolors.

#### Algorithm: Minimum Recolors to Get K Consecutive Black Blocks

**Problem:** Given a string `blocks` representing a sequence of blocks (where 'W' is white and 'B' is black), and an integer `k`, find the minimum number of recolors needed to have `k` consecutive black blocks.

**Algorithm:**

1.  **Initialization:**
    * Initialize `left` to 0. (`left` pointer of the sliding window)
    * Initialize `numWhites` to 0. (Count of white blocks in the current window)
    * Initialize `numRecolors` to a large value (e.g., infinity or maximum integer). (Stores the minimum recolors found so far)

2.  **Sliding Window Iteration:**
    * Iterate through the `blocks` string using a `right` pointer, from index 0 to the end.
    * For each `right` index:
        * If `blocks[right]` is 'W' (white), increment `numWhites`.
        * If the current window size is `k` (i.e., `right - left + 1 == k`):
            * Update `numRecolors` to the minimum of its current value and `numWhites`.
            * If `blocks[left]` is 'W' (white), decrement `numWhites`.
            * Increment `left` to slide the window forward.

3.  **Return Result:**
    * Return `numRecolors`.

**Implementation:**

```java
class Solution {

    public int minimumRecolors(String blocks, int k) {
        int left = 0, numWhites = 0, numRecolors = Integer.MAX_VALUE;

        // Move right pointer
        for (int right = 0; right < blocks.length(); right++) {
            // Increment numWhites if block at right pointer is white
            if (blocks.charAt(right) == 'W') {
                numWhites++;
            }

            // k consecutive elements are found
            if (right - left + 1 == k) {
                // Update minumum
                numRecolors = Math.min(numRecolors, numWhites);

                // Decrement numWhites if block at left pointer is white
                if (blocks.charAt(left) == 'W') numWhites--;

                // Move left pointer
                left++;
            }
        }
        return numRecolors;
    }
}
```

```py
class Solution:
    def minimumRecolors(self, blocks: str, k: int) -> int:
        left = 0
        num_whites = 0
        num_recolors = float("inf")

        # Move right pointer
        for right in range(len(blocks)):

            # Increment num_whites if block at right pointer is white
            if blocks[right] == "W":
                num_whites += 1

            # k consecutive elements are found
            if right - left + 1 == k:

                # Update minimum
                num_recolors = min(num_recolors, num_whites)

                # Decrement num_whites if block at left pointer is white
                if blocks[left] == "W":
                    num_whites -= 1

                # Move left pointer
                left += 1

        return num_recolors
```

### Complexity Analysis

Let N be the size of blocks.

**Time Complexity:** O(N)

The algorithm iterates through each element of blocks exactly once, performing constant-time operations on each element. Specifically, in each iteration, it performs arithmetic operations, whose time complexities are independent of the input size. Therefore, the overall time complexity is linear to the number of elements in blocks, O(n).

**Space Complexity:** O(1)

The space required does not depend on the size of the input value or any data structures that require additional space, so only constant O(1) space is used.