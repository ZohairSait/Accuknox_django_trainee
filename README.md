# Accuknox Django Demo


## Answers to Accuknox Trainee Questions

### Topic: Django Signals

#### Question 1: By default, are Django signals executed synchronously or asynchronously?
**Answer**: By default Django signals are **synchronous**. Basically, when a signal is sent, the connected receiver functions are executed immediately.

**Code Snippet**:  

```python

from django.db.models.signals import post_save
from django.dispatch import receiver
from django.contrib.auth.models import User
import time

@receiver(post_save, sender=User)
def my_signal_handler(sender, instance, created, **kwargs):
    print("Signal handler started")
    time.sleep(2)  
    print("Signal handler finished")

print("Before saving user")
user = User.objects.create(username="testuser")
print("After saving user")
```

**Test**: Once you run the above code, the below output will be displayed.
```
Before saving user
Signal handler started
Signal handler finished
After saving user
```

---

#### Question 2: Do Django signals run in the same thread as the caller?
**Answer**: Yup, Django signals run in the **same thread** as whatever triggered them by default.


```python
@receiver(post_save, sender=User)
def my_signal_handler(sender, instance, created, **kwargs):
    print(f"Signal handler thread ID: {threading.current_thread().ident}")

print(f"Caller thread ID: {threading.current_thread().ident}")
user = User.objects.create(username="testuser")
```

**Test**: Once you run the above code, the below output will be displayed.
```
Caller thread ID: 140735793123456
Signal handler thread ID: 140735793123456
```

---

#### Question 3: By default, do Django signals run in the same database transaction as the caller?
**Answer**: Yes, they run in the **same database transaction** by default. If the signal rollback, it can undo the caller’s work also.

```python
@receiver(post_save, sender=User)
def my_signal_handler(sender, instance, created, **kwargs):
    if created:
        instance.username = "modified_by_signal"
        instance.save()
        print("Signal updated username")

# Test code
try:
    with transaction.atomic():
        print("Creating user")
        user = User.objects.create(username="original")
        print(f"Username after signal: {User.objects.get(id=user.id).username}")
        raise Exception("Forcing rollback")
except Exception as e:
    print(f"Exception caught: {e}")

print(f"Username after rollback: {User.objects.get(username='original').username}")
```

**Test**: 
1. Visit `http://127.0.0.1:8000/create/`. The terminal shows:
   ```
Creating user
Signal updated username
Username after signal: modified_by_signal
Exception caught: Forcing rollback
Username after rollback: original
   ```

---

### Topic: Custom Classes in Python

#### Task: Create a `Rectangle` class
**Requirements**:
- Takes `length: int` and `width: int` when created.


**Solution**:  

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {'length': self.length}
        yield {'width': self.width}

if __name__ == "__main__":
    rect = Rectangle(5, 10)
    for side in rect:
        print(side)
```

**Test**: Run it with:
```bash
python rectangle.py
```
Output:
```
{'length': 5}
{'width': 10}
```
The rectangle.py file is also attached.