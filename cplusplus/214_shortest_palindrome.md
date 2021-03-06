Given a string S, you are allowed to convert it to a palindrome by adding characters in front of it. Find and return the shortest palindrome you can find by performing this transformation.

For example:

```
Given "aacecaaa", return "aaacecaaa".

Given "abcd", return "dcbabcd".
```

# Solution

http://jakeboxer.com/blog/2009/12/13/the-knuth-morris-pratt-algorithm-in-my-own-words/

Firstly, let me share my understanding of KMP algorithm.  
The key of KMP is to build a look up table that records the match result of __proper prefix__ and __preper postfix__.

* __Proper prefix__: All the characters in a string, with one or more cut off the end. “S”, “Sn”, “Sna”, and “Snap” are all the proper prefixes of “Snape”.
* __Proper suffix__: All the characters in a string, with one or more cut off the beginning. “agrid”, “grid”, “rid”, “id”, and “d” are all proper suffixes of “Hagrid”.

_String can be prefix and suffix of itself, but it is not a proper prefix and proper suffix._

Value in the table means the max len of matching substring that exists in both proper prefix and proper postfix.  
In the proper prefix this substring should starts from 0, while in the proper postfix this substring should ends at current index.  

For example, now we have a string "ababc"  
The KMP table will look like this:  

```
a b a b c

0 0 1 2 0
```

(Note: we will not match substring with itself, so we will skip index 0)

So how does this table help us search string match faster?

Well, the answer is if we are trying to match a char after proper postfix with target string and failed, then we can smartly shift the string, so that the matching string in proper prefix will replace proper postfix and now we can try to match the char after proper prefix with this char in target.

Take above string as an example.

Now we try to match string "ababc" with "abababc".

We will initially have match as below

```
a b a b a b c (string x)

a b a b c (string y)

0 1 2 3 4 5 6
```

We found char at index 4 does not match, then we can use lookup table and shift the string y wisely.
We found table[3] = 2, which means we can shift the string y rightward by 2, and still have same but shorter prefix before index 4, like this:

```
a b a b a b c (string x)
___a b a b c (string y)

0 1 2 3 4 5 6
```

If there is a long gap between prefix and postfix, this shift can help us save a lot of time.
In the brute force way, we cannot do that because we have no information of the string. We have to compare each possible pair of chars. While in kmp, we know the information of string y so we can move smartly. We can directly jump to the next possible matching pair while discard useless pair of chars.

We are almost done with KMP, but we still have one special case that needs to be taken care of.

Say now we have a input like this:

```
a a b a a a (input String)
0 1 2 3 4 5 (index)
0 1 0 1 2 ? (KMP table)
```

How should we build the KMP table for this string?

Say the pointer in prefix is "x", which is at index 2 now and the pointer in postfix is "y" which is at index 5 now. we need to match "b" pointed by x with "a" pointed by y. It is an unmatched pair, how should we update the cell?

Well, we really don't need to reset it to 0, that will make us skip a valid shorter matching substring "aa".
What we do now is just to shorten the length of substring by 1 unit and try to match a shorter substring "aa". This can be done by moving pointer x to the index recorded in [indexOf(x)-1] while keep pointer y stay still. This is because by following the value in KMP table we can always make sure previous part of prefix and postfix is matched even we have shorten their length, so we only need to care about the char after matched part in prefix and postfix.

Use above example:

Firstly we try to compare prefix "aab" with postfix "aaa", pointer in prefix now points to "b" while pointer in postfix now points to "a". So this means current len of postfix/prefix will not give a match, we need to shorten it.

So in the second step, we will fix pointer in postfix, and move pointer in prefix so that we can compare shorter prefix and postfix. The movement of pointer in prefix (say at index x) is done by using KMP table. We will set pointer in prefix to be table [indexOf(x)-1]. In this case, we will move prefix pointer to index 1. So now we try to compare prefix "aa" with postfix "aa".

Finally, we found the matching prefix and postfix, we just update the cell accordingly.

Above is my understanding of KMP algorithm, so how could we apply KMP to this problem

========================== I am just a splitter =================================

This problem asks us to add string before the input so the result string will be a palindrome.
We can convert it to an alternative problem"find the longest palindrome substring starts from index 0".
If we can get the length of such substring, then we can easily build a palindrome string by inserting the reverse part of substring after such substring before the original string.

Example:

input string:

```
abacd
```

longest palindrome substring starts from 0:

```
aba
```
Insert the reverse part of substring after palindrome substring before the head:

```
dcabacd
```

Now the problem becomes how to find the longest palindrome substring starts from 0.
We can solve it by using a trick + KMP.

The trick is to build a temp string like this:

```
s + "#" + reverse(s)
```

Then we run KMP on it, the value in last cell will be our solution. In this problem, we don't need to use KMP  
to match strings but instead we use the lookup table in KMP to find the palindrome.  

We add "#" here to force the match in reverse(s) starts from its first index  
What we do in KMP here is trying to find a match between prefix in s and a postfix in reverse(s). The match part will be palindrome substring.  

Example:  
input:  

```
catacb
```

Temp String:

```
catacb # bcatac
```

KMP table:

```
c a t a c b # b c a t a c

0 0 0 0 1 0 0 0 1 2 3 4 5
```




##### Solution 1

```cpp
class Solution {
public:
    string shortestPalindrome(string s) {
        string rev_s(s.rbegin(), s.rend());
        
        std::size_t index(0);
        for (std::size_t n = s.size(), len = n; len > 0; --len) {
            if (s.substr(0, len) == rev_s.substr(n - len)) {
                index = n - len;
                break;
            }
        }
        
        return rev_s.substr(0, index) + s;
    }
};
```

The problem with this solution is that it is not efficient enough.

Let's use N to represent the length of input string, then the for loop takes O(N). And inside the for loop, each substr function call takes worst O(N). So the time complexity is O(N^2).

##### Solution 2

For a string str, in worse case, the palindrome can be constructed by concatenate its reverse string with itself, ```rev_str + str```. 

For example, for string, 'aacd', we get its reverse string 'dcaa', and concatenate them as 'dcaaaacd'.

While in order to get the shortest palindrome, we need to find a way to get rid of the extra charaters in the middle.

And we find that the chars need to be removed is the longest common prefix and suffix if we construct a string as 
'aacd' + 'dcaa'. If we use KMP algorithm, we can get this length in linear time.

However, a special case is 'aa' whose reverse string is also 'aa', if we build the concatenated string 'aaaa', then the KMP algorithm will return length==3, since the longest common prefix and suffix string is 'aaa'.

To prevent that, we need a special char in the middle, like 'aa#aa'.

```cpp
class Solution {
public:
  string shortestPalindrome(string s) {
    string rever(s.rbegin(),s.rend());
    string mirror = s + '#' + rever;
    vector<int> kmp = get_kmp(mirror);
    int len = kmp[mirror.size()-1];
    return rever.substr(0,s.size()-len) + s;
  }
private:
  vector<int> get_kmp(const string & s) {
    const int N(s.size());
    vector<int> res(N,0);
    int pos(1),len(0);
    while(pos<N){
      if(s[pos] == s[len]) {
        res[pos] = len + 1;
        ++ pos;
        ++ len;
      } else {
        if(len==0) {
          res[pos] = 0;
          ++ pos;
        } else {
          len = res[len-1];
        }
      }
    }
    return res;
  }
};
```

##### Solution 3 : optimize on solution 2 

```cpp
class Solution {
public:
    string shortestPalindrome(string s) {
        std::string rev_s(s.rbegin(),s.rend());
        std::string ss(s + '#' + rev_s);
        std::size_t common_proper_prefix_suffix_len =  GetCommonProperPrefixSuffixLength(ss);
        return rev_s.substr(0, s.size() - common_proper_prefix_suffix_len) + s;
    }
    
private:
    std::size_t GetCommonProperPrefixSuffixLength(const std::string &str) {
        if (str.empty()) return 0;
        std::size_t prefix_index(0), suffix_index(1);
        const std::size_t str_size(str.size());
        std::vector<std::size_t> kmp_table(str_size, 0);
        while (suffix_index < str_size) {
            if (str[suffix_index] == str[prefix_index]) { // match
                kmp_table[suffix_index] = prefix_index + 1;
                ++ suffix_index, ++ prefix_index;
            } else { // mismatch
                
                // match failed, we try to match a shorter substring
                
                if (prefix_index != 0) {
                    prefix_index = kmp_table[prefix_index - 1];
                } else {
                    kmp_table[suffix_index] = 0;
                    ++ suffix_index;
                }
            }
        }
        
        return kmp_table[str_size - 1];
    }
};
```
