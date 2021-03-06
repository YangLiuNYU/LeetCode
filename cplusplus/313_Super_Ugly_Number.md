Write a program to find the nth super ugly number.

Super ugly numbers are positive numbers whose all prime factors are in the given prime list primes of size k. For example, [1, 2, 4, 7, 8, 13, 14, 16, 19, 26, 28, 32] is the sequence of the first 12 super ugly numbers given primes = [2, 7, 13, 19] of size 4.

Note:
(1) 1 is a super ugly number for any given primes.
(2) The given numbers in primes are in ascending order.
(3) 0 < k ≤ 100, 0 < n ≤ 106, 0 < primes[i] < 1000.
(4) The nth super ugly number is guaranteed to fit in a 32-bit signed integer.

# Solution

### A naive solution
Our naive solution works like this, begin from val = 1, keep increasing val by +1 and check whether it is an ugly number.
If it is, added to the ugly numbers vector. 

The drawback of this solution is that, we are using bruteforce method here and we checked many numbers that we should not bother to check. 

e.g. if the primes are {5, 117}, then why should we even check number 2,3,4,6,7.. ?

```cpp
class Solution {
public:
    bool is_ugly(int number, const vector<int>& primes) {
        if (number <= 0 ) return false;
        for (int prime : primes) {
            while (number % prime == 0) number /= prime;
        }
        
        return number == 1;
    }
    int nthSuperUglyNumber(int n, vector<int>& primes) {
        if (n <= 0) return 0;
        vector<int> ugly_numbers(1, 1);
        int val(2);
        while (ugly_numbers.size() < n) {
            if (is_ugly(val, primes)) {
                ugly_numbers.emplace_back(val);
            }
            ++ val;
        }
        
        return ugly_numbers.back();
    }
};
```

### Solutione one

How can we do better than solution 1? 

The keypoint here is that new prime number is generated by multiplying a prime with previous generated ugly number. 

__Time complexity ： O(n * number of primes)__
  
  
```cpp  
class Solution {
public:
  int nthSuperUglyNumber(int n, vector<int>& primes) {
    int primes_size(primes.size());
    vector<int> index(primes_size,0), ugly(n,INT_MAX);
    ugly[0] = 1;
    for (int i=1;i<n;++i) {
      for (int j=0;j<primes_size;++j) ugly[i] = std::min(ugly[i],primes[j]*ugly[index[j]]);
      for (int j=0;j<primes_size;++j) index[j]+=(ugly[i]==ugly[index[j]]*primes[j]? 1 : 0);
    }
    return ugly[n-1];
  }
};
```

An easy to understood version.

```cpp
class Solution {
public:
    int nthSuperUglyNumber(int n, vector<int>& primes) {
        const std::size_t primes_size(primes.size());
        // ugly vector represents the generated ugly sequence
        // ugly_indexes is a mapper <prime_index, ugly_index>
        std::vector<int> ugly(n, INT_MAX), ugly_indexes(primes_size, 0);
        ugly[0] = 1;
        
        for (int i = 1; i < n; ++i) {
            
            for (int prime_index = 0; prime_index < primes_size; ++ prime_index) {
                ugly[i] = std::min(ugly[i], ugly[ugly_indexes[prime_index]] * primes[prime_index]);
            }
            
            for (int prime_index = 0; prime_index < primes_size; ++ prime_index) {
                if (ugly[i] == ugly[ugly_indexes[prime_index]] * primes[prime_index]) {
                    ++ ugly_indexes[prime_index];
                }
            }
        }
        
        return ugly[n - 1];
    }
};
```
