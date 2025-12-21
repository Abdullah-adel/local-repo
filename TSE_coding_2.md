Got it. Below are **5 problems (same level as Problems 1–3)**. For each one, you get:

* **Gentle walkthrough (what the code does + streaming vs storing)**
* **One full code block** (single function + small helpers if needed)
* **Show cases (1–2 tests)**
* **Big-O (time & space)**

I’ll keep everything Python and interview-friendly.

---

## Problem A — Parse `key=value` logs and compute error rate per service

### Gentle walkthrough

* We **stream** the file line by line (do not load all lines).
* For each line, we extract `service` and `status`.
* We count:

  * total requests per service
  * 5xx errors per service
* Finally, compute error rate `% = errors / total * 100`.

### Code

```python
def error_rate_per_service(log_path: str):
    """
    Input lines like:
      ts=1703000001 service=auth status=503 path=/login

    Output:
      dict: service -> (total, errors_5xx, error_rate_percent)
    """
    totals = {}  # service -> total count
    errors = {}  # service -> 5xx count

    with open(log_path, "r", encoding="utf-8", errors="ignore") as f:
        for line in f:  # STREAMING: one line at a time
            line = line.strip()
            if not line:
                continue

            # Simple key=value parsing using split (no regex needed)
            # Example token: "service=auth"
            service = None
            status = None

            for token in line.split():
                if token.startswith("service="):
                    service = token.split("=", 1)[1]
                elif token.startswith("status="):
                    status = token.split("=", 1)[1]

            # Skip malformed lines
            if service is None or status is None:
                continue

            totals[service] = totals.get(service, 0) + 1
            if status.startswith("5"):
                errors[service] = errors.get(service, 0) + 1

    result = {}
    for service, total in totals.items():
        err = errors.get(service, 0)
        rate = (err / total) * 100.0
        result[service] = (total, err, rate)

    return result
```

### Show cases (how you’d verify)

* **Case 1:** service has mix of 2xx and 5xx
* **Case 2:** malformed line missing status should be ignored

Example (mentally):

* Lines:

  * `service=auth status=200 ...`
  * `service=auth status=503 ...`
    → total=2, errors=1, rate=50%

### Big-O

* **Time:** O(n * t) where n = lines, t = tokens per line (usually small) → effectively **O(n)**
* **Space:** O(s) where s = number of services

---

## Problem B — First non-repeating character in a string

### Gentle walkthrough

* This is not streaming from file; it’s a string input.
* We do **two passes**:

  1. count characters
  2. find first char whose count is 1

### Code

```python
def first_unique_char_index(s: str) -> int:
    """
    Return index of first character that appears exactly once.
    If none, return -1.
    """
    counts = {}

    # Pass 1: count each character
    for ch in s:
        counts[ch] = counts.get(ch, 0) + 1

    # Pass 2: find first with count == 1
    for i, ch in enumerate(s):
        if counts[ch] == 1:
            return i

    return -1
```

### Show cases

* `"google"` → `l` is unique first → index **4**
* `"aabbcc"` → none → **-1**

### Big-O

* **Time:** O(n)
* **Space:** O(k) (k distinct characters)

---

## Problem C — Two Sum (hashmap)

### Gentle walkthrough

* We scan the array once.
* For each number `x`, we check if we already saw `target - x`.
* If yes: we found the pair.
* If no: store `x -> index` in a hashmap.

### Code

```python
def two_sum(nums, target):
    """
    Return indices (i, j) such that nums[i] + nums[j] == target.
    Assume exactly one solution exists.
    """
    seen = {}  # value -> index

    for i, x in enumerate(nums):
        need = target - x

        # If we saw the needed number before, we can answer now
        if need in seen:
            return (seen[need], i)

        # Otherwise remember this number's index
        seen[x] = i

    # If not found (depends on problem statement), return None
    return None
```

### Show cases

* `nums=[2,7,11,15], target=9` → (0,1)
* `nums=[3,2,4], target=6` → (1,2)

### Big-O

* **Time:** O(n)
* **Space:** O(n)

---

## Problem D — Longest substring without repeating characters (sliding window)

### Gentle walkthrough (important)

We keep a “window” of characters with no repeats, using two pointers:

* `left` = start of current window
* `right` = current index we are scanning

We store the **last seen index** of each character in a hashmap.
If we see a repeated char inside our window:

* move `left` to `last_seen[char] + 1`

This is the same “sliding window” idea as the flapping services problem, but on a string.

### Code

```python
def length_of_longest_unique_substring(s: str) -> int:
    """
    Return length of the longest substring without repeating characters.
    """
    last_seen = {}  # char -> last index
    left = 0
    best = 0

    for right, ch in enumerate(s):
        # If char was seen and is inside current window, move left
        if ch in last_seen and last_seen[ch] >= left:
            left = last_seen[ch] + 1

        # Update last seen index
        last_seen[ch] = right

        # Update best length
        current_len = right - left + 1
        if current_len > best:
            best = current_len

    return best
```

### Show cases

* `"abcabcbb"` → longest `"abc"` → **3**
* `"bbbb"` → **1**
* `"pwwkew"` → longest `"wke"` → **3**

### Big-O

* **Time:** O(n) (each pointer moves forward only)
* **Space:** O(k) (distinct chars)

---

## Problem E — Valid parentheses / brackets (stack)

### Gentle walkthrough

* We push opening brackets onto a stack.
* When we see a closing bracket:

  * it must match the most recent opening bracket (top of stack)
* At the end, stack must be empty.

### Code

```python
def is_valid_brackets(s: str) -> bool:
    """
    Return True if brackets are valid and properly nested.
    Supports (), {}, [].
    """
    pairs = {")": "(", "}": "{", "]": "["}
    stack = []

    for ch in s:
        # Opening bracket: push
        if ch in "([{":
            stack.append(ch)
        # Closing bracket: must match stack top
        elif ch in pairs:
            if not stack:
                return False
            if stack[-1] != pairs[ch]:
                return False
            stack.pop()
        else:
            # If input may contain other chars, either ignore or return False.
            # We'll ignore non-bracket chars for robustness.
            continue

    return len(stack) == 0
```

### Show cases

* `"([]){}"` → True
* `"(]"` → False
* `"([)]"` → False
* `""` → True

### Big-O

* **Time:** O(n)
* **Space:** O(n) worst case (all opens)

---

## How to use these in your 60-minute Google code evaluation

Pick any one problem and follow this script:

1. Restate input/output + assumptions (30s)
2. Explain approach (30–60s)
3. Code (10–20 mins)
4. Run 1–2 show cases (2 mins)
5. Big-O (1 min)
6. Optional optimization (1 min)

If you want, I can now turn these + your Problems 1–3 into a **single “practice pack”** (8 problems) and tell you which to do in which order for 7 days.
