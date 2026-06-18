# Pattern 7: Parsing & Simulation

## Identify This Pattern
- "Decode a string / expression"
- "Evaluate a mathematical expression"
- "Convert between numeral systems (Roman, base-k)"
- "Implement strStr / atoi / basic calculator"
- "Reverse words, simplify path"
- Involves processing characters sequentially with state

---

## Templates

### Stack-based Decode (nested structures)
```cpp
// Decode string "3[a2[b]]" → "abbabbabb"
string decodeString(string s) {
    stack<int> counts;
    stack<string> strs;
    string cur = "";
    int k = 0;
    for (char c : s) {
        if (isdigit(c)) {
            k = k * 10 + (c - '0');
        } else if (c == '[') {
            counts.push(k); k = 0;
            strs.push(cur); cur = "";
        } else if (c == ']') {
            int repeat = counts.top(); counts.pop();
            string prev = strs.top(); strs.pop();
            while (repeat--) prev += cur;
            cur = prev;
        } else {
            cur += c;
        }
    }
    return cur;
}
```

### String to Integer (atoi) — Handle edge cases
```cpp
int myAtoi(string s) {
    int i = 0, n = s.size(), sign = 1;
    long result = 0;

    while (i < n && s[i] == ' ') i++;           // skip leading spaces
    if (i < n && (s[i] == '+' || s[i] == '-'))
        sign = (s[i++] == '+') ? 1 : -1;        // sign

    while (i < n && isdigit(s[i])) {
        result = result * 10 + (s[i++] - '0');
        if (result * sign > INT_MAX) return INT_MAX;
        if (result * sign < INT_MIN) return INT_MIN;
    }
    return (int)(result * sign);
}
```

### Simplify Path (Unix filesystem)
```cpp
string simplifyPath(string path) {
    vector<string> parts;
    stringstream ss(path);
    string token;
    while (getline(ss, token, '/')) {
        if (token == "" || token == ".") continue;
        if (token == "..") { if (!parts.empty()) parts.pop_back(); }
        else parts.push_back(token);
    }
    string res = "/";
    for (int i = 0; i < parts.size(); i++) {
        if (i > 0) res += "/";
        res += parts[i];
    }
    return res;
}
```

### Roman to Integer
```cpp
int romanToInt(string s) {
    unordered_map<char,int> val = {
        {'I',1},{'V',5},{'X',10},{'L',50},
        {'C',100},{'D',500},{'M',1000}
    };
    int result = 0;
    for (int i = 0; i < s.size(); i++) {
        if (i+1 < s.size() && val[s[i]] < val[s[i+1]])
            result -= val[s[i]];  // subtractive case
        else
            result += val[s[i]];
    }
    return result;
}
```

### Integer to Roman
```cpp
string intToRoman(int num) {
    vector<pair<int,string>> vals = {
        {1000,"M"},{900,"CM"},{500,"D"},{400,"CD"},
        {100,"C"},{90,"XC"},{50,"L"},{40,"XL"},
        {10,"X"},{9,"IX"},{5,"V"},{4,"IV"},{1,"I"}
    };
    string res = "";
    for (auto& [v, sym] : vals) {
        while (num >= v) { res += sym; num -= v; }
    }
    return res;
}
```

---

## Problems

### Easy
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [13](https://leetcode.com/problems/roman-to-integer/) | Roman to Integer | If current < next → subtract, else add | Process left to right, compare each char to next |
| [12](https://leetcode.com/problems/integer-to-roman/) | Integer to Roman | Greedy: repeatedly subtract largest fitting value | Pre-define all 13 value-symbol pairs including subtractive forms |
| [14](https://leetcode.com/problems/longest-common-prefix/) | Longest Common Prefix | Compare char by char against first string | Stop when any string differs or ends |
| [171](https://leetcode.com/problems/excel-sheet-column-number/) | Excel Sheet Column Number | Base-26 where A=1 | `result = result * 26 + (c - 'A' + 1)` |
| [168](https://leetcode.com/problems/excel-sheet-column-title/) | Excel Sheet Column Title | Reverse of above. But it's 1-indexed (no zero) | `num--` before each `% 26` to shift to 0-indexed |
| [67](https://leetcode.com/problems/add-binary/) | Add Binary | Add from right, carry = sum / 2 | Use carry variable, build result backwards |

### Medium
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [8](https://leetcode.com/problems/string-to-integer-atoi/) | String to Integer (atoi) | 4 steps: skip spaces, parse sign, parse digits, clamp | Use `long` to detect overflow before clamping |
| [394](https://leetcode.com/problems/decode-string/) | Decode String | Stack for counts and strings. Process `[` and `]` | Two stacks: one for repeat counts, one for string prefixes |
| [71](https://leetcode.com/problems/simplify-path/) | Simplify Path | Split by `/`, process `.` and `..` with a stack | `getline(ss, token, '/')` splits by delimiter |
| [43](https://leetcode.com/problems/multiply-strings/) | Multiply Strings | Grade-school multiplication. Result[i+j] += d1*d2 | Handle carry after all multiplications. Trim leading zeros. |
| [151](https://leetcode.com/problems/reverse-words-in-a-string/) | Reverse Words in a String | Trim, split, reverse order OR: reverse all then reverse each word | Edge: multiple spaces between words, leading/trailing spaces |
| [227](https://leetcode.com/problems/basic-calculator-ii/) | Basic Calculator II | Stack. For +/-: push, for */: pop and compute | Process each number when you see the NEXT operator |
| [224](https://leetcode.com/problems/basic-calculator/) | Basic Calculator | Stack for sign and parenthesis context | Push current result and sign when `(`, pop and combine on `)` |

### Hard
| # | Problem | Hint | Key Insight |
|---|---------|------|-------------|
| [65](https://leetcode.com/problems/valid-number/) | Valid Number | State machine / track what's been seen | Tricky edge cases: `e/E`, signs, decimal point positions |
| [273](https://leetcode.com/problems/integer-to-english-words/) | Integer to English Words | Recursive: handle groups of 3 digits | Define billion/million/thousand/hundred separately |
| [68](https://leetcode.com/problems/text-justification/) | Text Justification | Greedy: fill as many words as fit, distribute spaces | Extra spaces go left-first; last line is left-justified |

---

## Basic Calculator — Key Insight

```
For +/-: push value to stack with appropriate sign
For */÷: pop from stack, apply operation, push result back
At end: sum the stack

Why this works: */÷ has higher precedence.
By computing them immediately (not deferring), we respect operator precedence.
When we push for +/-, we're deferring until we see the full expression context.
```

## Decode String — Two Stacks Pattern

```
Stack 1: counts   (how many times to repeat)
Stack 2: strings  (string built so far before this bracket)

On '[': save current state (push cur string and count)
On ']': restore state (pop count and prev string, repeat and concat)
```

## Common Mistakes

- atoi: not using `long` during accumulation → INT_MAX overflow before clamping
- Decode String: multi-digit repeat counts — `k = k*10 + digit`, not just `k = digit`
- Simplify Path: not handling edge case where path ends with `/` → produces empty token
- Roman to Integer: checking `val[s[i]] < val[s[i+1]]` — need `i+1 < s.size()` bounds check

## Follow-up Interviewers Love

- "atoi — what are the edge cases?" → Leading spaces, sign, overflow (INT_MIN/MAX), non-digit characters after digits
- "Decode String — what if brackets can be nested 1000 levels deep?" → Stack depth = nesting level; O(output_length) time
- "Basic Calculator — how do you add parentheses support?" → Push current result and sign on `(`, pop and combine on `)`
