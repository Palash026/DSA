# 2523. Closest Prime Numbers in Range

**Topic**: Math, Number Theory

Given two positive integers `left` and `right`, find the two integers `num1` and `num2` such that:

- `left <= num1 < num2 <= right`
- Both `num1` and `num2` are prime numbers.
- `num2 - num1` is the minimum amongst all other pairs satisfying the above conditions.

Return the positive integer array `ans = [num1, num2]`. If there are multiple pairs satisfying these conditions, return the one with the smallest `num1` value. If no such numbers exist, return `[-1, -1]`.

## Examples

### Example 1

**Input**: `left = 10`, `right = 19`  
**Output**: `[11, 13]`  
**Explanation**: The prime numbers between 10 and 19 are 11, 13, 17, and 19. The closest gap between any pair is 2, which can be achieved by `[11, 13]` or `[17, 19]`. Since 11 is smaller than 17, we return the first pair.

### Example 2

**Input**: `left = 4`, `right = 6`  
**Output**: `[-1, -1]`  
**Explanation**: There exists only one prime number in the given range, so the conditions cannot be satisfied.

## Constraints

- `1 <= left <= right <= 10^6`

## Solution

### Approach 1: Sieve of Eratosthenes

**Intuition**

We are given two numbers, `left` and `right`, and we need to find a pair of prime numbers within this range such that their difference is minimized. If multiple pairs have the same minimum difference, we return the one with the smallest values. If no such pair exists, we return `[-1, -1]`.

A simple approach would be to iterate through all numbers in this range, check whether each number is prime, store the primes, and then determine the pair with the smallest difference. However, checking if a number is prime requires verifying that it has no divisors other than 1 and itself. A naive way to do this is to test divisibility for all numbers up to `n`, but a more optimized approach would only check divisibility up to `sqrt(n)`. Even with this optimization, the approach remains too slow. Since `right` can be as large as `10^6`, iterating through all numbers and performing a divisibility check for each would still be inefficient, leading to a Time Limit Exceeded (TLE) error.

A much faster way to find all prime numbers up to a given limit is the Sieve of Eratosthenes. Instead of checking each number one by one, the sieve marks multiples of each prime in bulk, eliminating the need for repeated divisibility checks.

We start with a list of numbers from 2 to 100. Notice we skip 1 since it’s not considered a prime. Starting with the smallest prime, 2, we know it’s prime because it hasn’t been marked yet. So, we keep it. Now, we cross out all multiples of 2 (like 4, 6, 8, etc.) because they’re definitely not prime. The next number that isn’t crossed out is 3, so we mark it as a prime. Then, we cross out all multiples of 3 (like 6, 9, 12, etc.). We keep going, finding the next unmarked number (which will be 5), and marking all of its multiples. We do this for 7 as well and continue until we’ve processed all numbers up to the limit.

The beauty of the Sieve of Eratosthenes is that it saves a lot of time by marking off composites in bulk, rather than testing each number individually to see if it’s prime. By the end, any number that’s still unmarked is a prime.

As we proceed, we collect all the numbers in an array `primeNumbers`, where `sieve[prime] = 1`. For any marked (non-prime) number, we could also keep track of the specific prime that marked it, though, for this problem, it’s sufficient to identify which numbers are prime.

Since all values lie between 1 and 1000000, we can iterate through the array, check for the minimum difference between two consecutive primes, and return it as the answer.

## Algorithm

### Main Function: `closestPrimes(int left, int right)`

1. **Generate Prime Numbers using Sieve:**
    - Create an integer array `sieve` of size `(right + 1)`, initialized to 1 (indicating prime numbers).
    - Set `sieve[0]` and `sieve[1]` to 0 (since 0 and 1 are not prime).
    - Iterate through numbers from 2 to `sqrt(right)`:
      - If the number is marked as prime (`sieve[number] == 1`), mark all its multiples as non-prime (`sieve[multiple] = 0`).

2. **Collect Prime Numbers in Range:**
    - Create a vector `primeNumbers` to store prime numbers within `[left, right]`.
    - Iterate through numbers from `left` to `right`:
      - If `sieve[num] == 1`, add `num` to `primeNumbers`.

3. **Find the Closest Prime Pair:**
    - If `primeNumbers.size() < 2`, return `{-1, -1}` (since there are not enough primes).
    - Initialize `minDifference` to the maximum integer value and `closestPair` to `{-1, -1}`.
    - Iterate through `primeNumbers` and check consecutive primes:
      - Compute `difference = primeNumbers[index] - primeNumbers[index - 1]`.
      - If `difference` is smaller than `minDifference`, update `closestPair = {primeNumbers[index - 1], primeNumbers[index]}`.
    - Return `closestPair` as the result.

### Helper Function: `sieve(int upperLimit)`

1. Create an integer vector `sieve` of size `(upperLimit + 1)`, initialized to 1 (indicating prime numbers).
2. Set `sieve[0]` and `sieve[1]` to 0 (since 0 and 1 are not prime).
3. Iterate through numbers from 2 to `sqrt(upperLimit)`:
    - If `sieve[number] == 1`, mark all multiples of `number` as 0 (non-prime).
4. Return the `sieve` array.

### Implementation
```py
class Solution:
    def _sieve(self, upper_limit):
        # Create an integer list to mark prime numbers (True = prime, False = not prime)
        sieve = [True] * (upper_limit + 1)
        sieve[0] = sieve[1] = False  # 0 and 1 are not prime

        for number in range(2, int(upper_limit**0.5) + 1):
            if sieve[number]:
                # Mark all multiples of 'number' as non-prime
                for multiple in range(number * number, upper_limit + 1, number):
                    sieve[multiple] = False
        return sieve

    def closestPrimes(self, left, right):
        # Step 1: Get all prime numbers up to 'right' using sieve
        sieve_array = self._sieve(right)

        prime_numbers = [
            num for num in range(left, right + 1) if sieve_array[num]
        ]

        # Step 2: Find the closest prime pair
        if len(prime_numbers) < 2:
            return -1, -1  # Less than two primes

        min_difference = float("inf")
        closest_pair = (-1, -1)

        for index in range(1, len(prime_numbers)):
            difference = prime_numbers[index] - prime_numbers[index - 1]
            if difference < min_difference:
                min_difference = difference
                closest_pair = prime_numbers[index - 1], prime_numbers[index]

        return closest_pair
```

```java
class Solution {

    public int[] closestPrimes(int left, int right) {
        // Step 1: Get all prime numbers up to 'right' using sieve
        int[] sieveArray = sieve(right);

        List<Integer> primeNumbers = new ArrayList<>(); // Store all primes in the range [left, right]
        for (int num = left; num <= right; num++) {
            // If number is prime add to the primeNumbers list
            if (sieveArray[num] == 1) {
                primeNumbers.add(num);
            }
        }

        // Step 2: Find the closest prime pair
        if (primeNumbers.size() < 2) return new int[] { -1, -1 }; // Less than two primes available

        int minDifference = Integer.MAX_VALUE;
        int[] closestPair = new int[2];
        // setting initial values
        Arrays.fill(closestPair, -1);

        for (int index = 1; index < primeNumbers.size(); index++) {
            int difference =
                primeNumbers.get(index) - primeNumbers.get(index - 1);
            if (difference < minDifference) {
                minDifference = difference;
                closestPair[0] = primeNumbers.get(index - 1);
                closestPair[1] = primeNumbers.get(index);
            }
        }

        return closestPair;
    }

    private int[] sieve(int upperLimit) {
        // Initiate an int array to mark prime numbers (1 = prime, else it's not)
        int[] sieve = new int[upperLimit + 1];
        // Assuming all numbers as prime initially
        Arrays.fill(sieve, 1);

        // 0 and 1 are not prime
        sieve[0] = 0;
        sieve[1] = 0;

        for (int number = 2; number * number <= upperLimit; number++) {
            if (sieve[number] == 1) {
                // Mark all multiples of 'number' as non-prime
                for (
                    int multiple = number * number;
                    multiple <= upperLimit;
                    multiple += number
                ) {
                    sieve[multiple] = 0;
                }
            }
        }
        return sieve;
    }
}
```

### Complexity Analysis
Let R be `right` and L be `left`, representing the range within which we search for prime numbers.

- Time Complexity: O(Rlog(log(R))+R−L)

    The Sieve of Eratosthenes runs in O(Rlog(log(R))), where R is the upper limit of the sieve. After generating the sieve, iterating through the range [L,R] to collect prime numbers takes O(R−L). Finally, finding the closest prime pair requires O(R−L) operations.

    Thus, the overall time complexity is O(Rlog(log(R))+R−L).

- Space Complexity: O(R)

    The algorithm uses a `sieve` array of size O(R) to mark prime numbers. Additionally, the vector storing prime numbers within the range [L,R] can have at most O(R−L) elements. Thus, the overall space complexity is O(R).
---
