Given a string s, return all the palindromic permutations (without duplicates) of it. Return an empty list if no palindromic permutation could be form.

For example:

```
Given s = "aabb", return ["abba", "baab"].

Given s = "abc", return [].
```

Hint:

* If a palindromic permutation exists, we just need to generate the first half of the string.  
* To generate all distinct permutations of a (half of) string, use a similar approach from: Permutations II or Next Permutation.  

# Solution

As suggested by the first hint, we just need to get one half of the string (each character appears half the times in s), then generate all its unique permutations and concatenate them with the reversed half (possibly the single middle character if the length of the string s is odd).

All the above work will only be done if an palindrome permutation exists. To tell whether a palindrome permutation exists, Palindrome Permutation has paved the way for us. To generate all the unique permutations, you may as well refer to Permutations II or Next Permutation as suggested by the second hint. But I guess this part is not the main point of this problem, so I directly use the next_permutation of C++. Well, I am not quite whether this is the right way, but this gives shorter codes. Moreover, the tag of this problem is backtracking, which I guess only needs to be used in generating the permutations. After this is done, we can simply concatenate to make the palindromes.

```cpp
class Solution {
public:
    vector<string> generatePalindromes(string s) {
        vector<string> palindromes;
        unordered_map<char, int> counts;
        for (char c : s) counts[c]++;
        int odd = 0; char mid; string half;
        for (auto p : counts) {
            if (p.second & 1) {
                odd++, mid = p.first;
                if (odd > 1) return palindromes;
            }
            half += string(p.second / 2, p.first);
        }
        palindromes = permutations(half);
        for (string& p : palindromes) {
            string t(p);
            reverse(t.begin(), t.end());
            if (odd) t = mid + t;
            p += t;
        }
        return palindromes;
    }
private: 
    vector<string> permutations(string& s) {
        vector<string> perms;
        string t(s);
        do {
            perms.push_back(s);
            next_permutation(s.begin(), s.end());
        } while (s != t);
        return perms;
    }
};
```

