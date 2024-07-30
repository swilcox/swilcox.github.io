+++
title = 'Gonna Deque You'
date = 2024-07-30T08:35:27-05:00
draft = false
tags = ["python", "deque", "stack"]
summary = "deque vs. lists in python for a stack"
+++
# Gonna Deque You

## Terminology Problems

I was once presented a coding puzzle that called for a stack. Being the terminology-challenged person that I am, I pictured the structure that was needing. I so items being pushed on and off of a stack but I used the word "queue" when describing what I was going to use. When asked to clarify, I essentially doubled down on the word queue but clarified that I needed a LIFO (last-in-first-out) queue. Finally, after being nudged further I realized I was being prompted for the name "stack". Duh! Again, terminology issues. I can picture what is needed.

So because of what I was picturing and because I knew I wasn't concerned about getting random elements in the "stack", I actually thought to use the python [deque](https://docs.python.org/3/library/collections.html#collections.deque) structure. Even the documentation says "Deques are a generalization of stacks and queues...short for *double-ended queue*". I feel somewhat vindicated for conflating the terms. And I recalled some comments from Raymond Hettinger about lists v deque.

A quick search lead me to [this stackoverflow question/answer](https://stackoverflow.com/questions/23487307/python-deque-vs-list-performance-comparison). And upon my own very unscientific hacking around, I came to a couple of conclusions.

## Using a Deque for a Stack

You can literally interchange in code a `deque` for a `list` in python when implementing a stack.

### list

```python
my_stack = []
my_stack.append("first item")
my_stack.append("second item")
print(my_stack[-1])  # can check the top of the stack --> "second item"
my_stack.pop()  # pop from the top of the stack --> "second item"
len(my_stack)  # 1 still in the stack
```

All of the operations on the stack are essentially O(1) because we're appending onto the end of the list and removing from the end of the list. Because of the implementation of lists in Python (again, go read Raymond's comments linked above), there are some `realloc` moments happening in chunks as the list grows. So it's still averaging O(1), but in theory, some `append` operations will take more time and effort than others.

### deque

```python
from collections import deque

my_stack = deque()
my_stack.append("first item")
my_stack.append("second item")
print(my_stack[-1])  # can check the top of the stack --> "second item"
my_stack.pop()  # pop from the top of the stack --> "second item"
len(my_stack)  # 1 still in the stack
```

All of the operations on the stack (in this case a `deque`) are essentially also O(1). In this case, it's doubly linked list, and we could add onto the bottom or left (however you'd like to picture it) *also* at O(1) and likewise, we could pop from the bottom or left if we wanted to at O(1). Allocations on append should be more consistent. What we lose is O(1) access to individual elements.

### My Own Unscientific Comparisons

I did some timing experiments with many smallish stacks and a few very large stacks, and pretty much saw no significant difference in speed between either approach when used purely as a stack. I guess that's to be expected because if you're really just using it as a normal "stack" (look at me with *terminology*) then you're not leveraging any of the benefits of a double-ended queue. All that's left is the `realloc` moments of the dynamic array powering the `list`. And that's been so refined (as of python 3.12) that it's not noticeable (to me in my crude experiments). Also, for very small examples, the `deque` seems to have more memory overhead for the structure as compared to a `list`. Again, that makes sense when you think about a python `list` just being an array. As things grow, you would expect the `list` would grow in memory usage in chunks and the `deque` would grow linearly with the elements added to it. So I suppose if you were dealing with super large stacks (or queues :wink:), then the more predictable memory usage could be an asset. But if you're doing super large uses of memory like that, you probably have other worries.

## Conclusion

When implementing a traditional *stack* it seems better to just use a `list`. You'll confuse fewer people and you won't get crap for conflating proper terminology. But hey?! I'm probably wrong so feel free to ignore this.