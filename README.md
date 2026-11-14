# Library Manager

A small command-line program for a campus library that lends books, magazines and DVDs. Members can borrow and return items, late fees differ per item type, and invalid actions are refused with clear messages instead of crashing.

## How to run

```bash
python main.py      # Windows
python3 main.py     # macOS / Linux
```

Requires Python 3 only; there are no external dependencies.

## Files

| File | Contents |
|------|----------|
| `models.py` | `LibraryItem` (abstract parent), `Book`, `Magazine`, `DVD`, and `Member` |
| `main.py` | Demo that exercises every feature and prints the results |

## Features

| Feature | Where |
|---------|-------|
| Abstract parent class | `LibraryItem` declares `late_fee()` and `describe()` with `@abstractmethod` |
| Inheritance | `Book`, `Magazine` and `DVD` call `super().__init__(title, item_id)` and add one field each |
| Polymorphism | Each child implements `late_fee()` with its own daily rate (Rs 2, Rs 1, Rs 5) |
| Encapsulation | `__available` and `__borrowed` are private and changed only through methods |
| Class attribute | `LibraryItem.total_items` counts every item created |
| Default argument | `Member(name, member_id, max_items=3)` |
| Exceptions | `borrow()` and `return_item()` raise `ValueError` with a clear message |

## Sample output

```
--- 1. Borrowing ---
Alice borrowed: Book: Python 101 by A. Rao (ID 1)
Could not borrow: 'Python 101' is already borrowed

--- 2. Limit and return errors ---
Bob borrowed: Magazine: Tech Today, issue 42 (ID 2)
Could not borrow: Bob has reached the borrow limit of 1 item(s)
Could not return: Alice has not borrowed 'Inception'
Alice has: ['Python 101']
Bob has: ['Tech Today']
Alice returned 'Python 101' 4 days late, fee: Rs 8
Bob returned 'Tech Today' on time, fee: Rs 0
Alice has: []

--- 3. Polymorphism ---
Book: Python 101 by A. Rao (ID 1) -> late fee for 3 days: Rs 6
Magazine: Tech Today, issue 42 (ID 2) -> late fee for 3 days: Rs 3
DVD: Inception, 148 min (ID 3) -> late fee for 3 days: Rs 15

--- 4. Class attribute ---
Total items created: 3

--- 5. Abstraction ---
Cannot create a LibraryItem: Can't instantiate abstract class LibraryItem without an implementation for abstract methods 'describe', 'late_fee'

--- 6. Encapsulation ---
Direct access failed: 'Book' object has no attribute '__available'
Proper access via is_available(): True
```

## Reflection

### 1. Inheritance and abstraction: why is `LibraryItem` an abstract class?

Every item has a title, an ID and an availability flag, and every item can be borrowed and returned. That shared behaviour lives once in `LibraryItem`, so `Book`, `Magazine` and `DVD` only add what is different about them: one extra field and a daily rate.

`LibraryItem` is abstract because a plain "library item" does not exist in the library and has no late fee of its own. Marking `late_fee()` with `@abstractmethod` means Python refuses to create a `LibraryItem` directly (it raises a `TypeError`) and also refuses to create any child class that forgets to implement `late_fee()`. The mistake is caught when the object is created, not later when a fee is calculated.

### 2. Polymorphism: how does the fee loop work without checking the type?

`main.py` puts a book, a magazine and a DVD in one list and calls `item.late_fee(3)` on each. The loop contains no `if` on the type. Python looks up `late_fee()` on whatever object it is given, so the same call returns Rs 6, Rs 3 and Rs 15.

The benefit is that new item types need no changes to existing code. Adding an `AudioBook` class with its own rate would work in the same loop, and in `Member.return_item()`, without editing either.

### 3. Encapsulation and error handling: why private attributes and exceptions?

`__available` and `__borrowed` are private because they must stay consistent with each other. If outside code could set `book.__available = True` directly, the book could be marked available while still on a member's list. Because the only way to change them is through `borrow()` and `return_item()`, the checks always run first. Python enforces this through name mangling: `book.__available` from outside the class raises an `AttributeError`, and `is_available()` is the proper way to read it. `borrowed_titles()` returns a new list of titles, so the caller cannot modify the real list.

Invalid actions raise a `ValueError` with a message that says what went wrong, for example `'Python 101' is already borrowed`. An exception cannot be silently ignored the way a returned `False` can, and it lets `main.py` decide how to report the problem. Each risky call is wrapped in `try / except ValueError as e`, so the program prints a friendly message and keeps running.
