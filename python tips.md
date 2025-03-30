# python tips

1.

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
filtered = []
for n in numbers:
    if n % 2 == 0:
        filtered.append(n)
multiplied = []
for n in filtered:
    multiplied.append(n * 10)
added = []
for n in multiplied:
    added.append(n + 5)
total = 0
count = 0
for n in added:
    total += n
    count += 1
average = total / count
print(average)
```

2.

```python
numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
average = sum(map(lambda n: n + 5, map(lambda n: n * 10, filter(lambda n: n % 2 == 0, numbers)))) / len(numbers)
print(average)
```

3.

