---
layout: 
title: Algorithm-Palindrome Number
date: 2020-04-29 23:45:31
tags: "Algorithm"
---

## Description
Determine whether an integer is a palindrome. An integer is a palindrome when it reads the same backward as forward.

## Examples
```
Input: -1
Output: false
```
```
Input: 101
Output: true
```
```
Input: 20
Output: false
```

## Pseudocode
Actually, the first thought rising up in my mind was to convert the integer number into string, then compare the characters. However, there is a follow up requirement, which is `Coud you solve it without converting the integer to a string?`. Then I thought of another way: get each digit of the integer, store them into a array list, then compare these digits.

## Implementation-Java
```
import java.util.List;
import java.util.ArrayList;

public class Solution{
    public boolean isPalindrome(int x) {

        if(x < 0){
            return false;
        }
        if(x == 0) {
            return true;
        }
        // split each digit of the number x
        List<Integer> numberList = new ArrayList<>();
        int remainder = 0;
        int divideNumber = 10;
        for(;;){
            remainder = x % divideNumber;
            x = x / divideNumber;
            numberList.add(remainder);
            if(x == 0){
                break;
            }
        }
        // compare forward digits with backward digits
        boolean flag = true;
        for(int i=0, j= numberList.size() - 1; i < j; i++, j--){
            if(numberList.get(i) != numberList.get(j)){
                flag = false;
                break;
            }
        }
        return flag;
    }
}
```
## Improved Implementation
The improved algorithm is from the [online discussion](https://leetcode.com/problems/palindrome-number/discuss/599060/Super-simple-java-solution-without-changing-to-String). It is a little different from my solution but the general idea is the same. This implement directly calculate the reversed number of the original number, which makes the logic more clearly.
```
public class Solution{
    public boolean isPalindrome(int x) {
        int original = x;
        
        if(x < 0){
            return false;
        }
        
        int reverse = 0;
        while(x >= 1){
            int digit = x % 10;
            reverse = reverse * 10 + digit;
            x = x / 10;
        }
        
        if(reverse == original){
            return true;
        }else{
            return false;
        }
    }
}
```

## References

[Palindrome Numbers](https://leetcode.com/problems/palindrome-number/)