## Django Singals

**Answer 1:**

By Default Django Signals are executed synchronously

```
#respective imports
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver
import time

# example User model
class User(models.Model):
    name = string = models.CharField(max_length=100)

# receiver function with User signal
@receiver(post_save, sender=User)
def user_signal_handler(sender, instance, **kwargs):
    print(f"Signal handler started at {time.time()}")
    time.sleep(5)
    print(f"Signal handler finished at {time.time()}")

# a function based view or a management command to test
def create_user():
    print(f"Going to create user at {time.time()}")
    MyModel.objects.create(name="Test")
    print(f"User created at {time.time()}")
```

**Answer Explanation:**

When we create the model, the signal is fired and the receiver function, Signal handler prints the start time, then it sleeps for 5 seconds, after that signal handle finished time is printed. Which concludes that by default Singals are synchrounous. otherwise the finished time could have printed before.

**Answer 2:**

Yes, Django signals run in the same thread as the caller

```
#respective imports
import threading
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver

# example User model
class User(models.Model):
    name = string = models.CharField(max_length=100)

# receiver function with User signal
@receiver(post_save, sender=User)
def user_signal_handler(sender, instance, **kwargs):
    print(f"Signal handler running in thread: {threading.current_thread().name}")

# a function based view or a management command to test
def create_user():
    print(f"Creating user in thread: {threading.current_thread().name}")
    User.objects.create(name="Manish")
```

**Answer Explanation:**

When we run the create_user() function we see that both create_user() which is caller here and user_signal_handler(), which is receiver function both runs in the same thread.

**Answer 3:**

Yes, by default django signals run in the same database transaction as the caller.

```
#respective imports
import threading
from django.db import models, transaction
from django.db.models.signals import post_save
from django.dispatch import receiver

class Author(models.Model):
    name = models.CharField(max_length=100)

class Book(models.Model):
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    info = models.CharField(max_length=100)

@receiver(post_save, sender=Author)
def author_signal_handler(sender, instance, **kwargs):
    Author.objects.create(author=instance, info="Created by signal")

# In a view or management command:
def create_book():
    try:
        with transaction.atomic():
            author = Author.objects.create(name="manish")
        # Simulated an error to trigger roll-back
            raise Exception("Simulated error")
    except Exception as e:
        print(f"Error occurred: {e}")

    # Checking here, if the Book object was created
    book_exists = Book.objects.filter(author__name="manish").exists()
    print(f"Book model exists: {book_exists}")
```

**Answer Explanation:**

When create_book() function is run, it creates an Author model instance
which triggers the creation of Book model instance. So we forcefully simulate an error here, so that it rolls back the transaction, as atomic transaction is used here. So when we check whether the book model exists it prints that it doesnt exists. It proves that the signal ran whithin the same transaction

## Custom Classes in Python

See the file name custom_classes.py
