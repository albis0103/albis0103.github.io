# [#125] Valid Palindrome

- DS or Algo: stack, string process
- Date: 07/29
- Status: AC 

## What went wrong
string process(isalnum(), lower())
(skip if solved cleanly)

## Problem
convert all letter from upper to lower, and remove all non-alphanumeric character
check Is it palindrome?
## Code

```python
def isPalindrome(self, s: str) -> bool:
	# string preprocess
	letter = ''
	for char in s:
		if char.isalnum():
			letter += char.lower()
	
	# palindrome process (stack)
	stack, n = [], len(letter)
	for i in range(n//2):
		stack.append(letter[i])
	start = n//2+(1 if n % 2 == 1 else 0)
	for j in range(start, n):
		if stack.pop() != letter[j]:
			return False
	return True
```
Java: String Builder!!!
```Java

public boolean isPalindrome(String s) {
	//string process
	StringBuilder sb = new StringBuilder();
	for(char c : s.toCharArray()){
		if(Character.isLetterOrDigit()){
			sb.append(Character.toLowerCase(c));
		}
	}
	String letter = sb.toString();
	
	int start = 0;
	int n = letter.length();
	ArrayDeque<Character> stack = new ArrayDeque<>();
	for(start = 0; start < n/2; start++){
		stack.push(letter.charAt(start));
	}
	if(n % 2 == 1){
		start ++;
	}
	for(int j = start; j < n; j++){
		if(stack.pop() != letter.charAt(j)){
			return false;
		}
	}
	return true;
}
```
## Redo?