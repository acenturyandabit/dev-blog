# Python quirks

## Logging Quirks

```py
# Level 1: I need to log something and this is a pre-production app and I don't care about pipe-based composition with other commands
print("a debug log")

# Level 2: I want to compose my cli app with other cli tools
import sys
print("the actual output")
print("some debug output", file=sys.stderr)

# Level 3: I want a proper logger with module names, and enable my logs based on
# an environment variable
import logging
logger = logging.getLogger(__name__)
logging.basicConfig(level=os.environ["LOGLEVEL"])
# subsequently, set LOGLEVEL=INFO in your environment to get logs
logger.info('Doing something')

# Level 4: I want to selectively enable logging on various modules
import logging
import sys
logger = logging.getLogger(__name__)
logger.setLevel(os.environ[__name__ + "_LOGLEVEL"])
print (f"Logger for {__name__} available, set "
         "{__name__}_LOGLEVEL variable to enable it",
         file=sys.stderr)
logger.info('Doing something')
```

If you're trying to make a python app that emits a large string without newlines, and the results seem to get clipped, try using `print("...", flush=True)` or `PYTHONUNBUFFERED=1` in your environment before running it. See <https://realpython.com/python-flush-print-output/>

## @staticmethod and @classmethod

A static method is a method that belongs to a class but doesn't use its `self` parameter. You can instantiate it using the `@staticmethod` decorator.

A class method is like a static method in that it doesn't get passed the `self` parameter either, but instead gets passed a `cls` method which is the class when the method is invoked. You can instantiate it using the `@classmethod` decorator.

Why would you use a class method if you can't access `self`, and you can access the class anyway from the scope of a static method? The answer is Runtime Polymorphism, a common concept in C++ but less explicitly mentioned in python. Check out this example:

```py
class BaseAnimal():
    name: str = "animal"
    
    @staticmethod
    def speak_static():
        print(f"I am an {BaseAnimal.name}")
    
    @classmethod
    def speak_class(cls):
        print(f"I am actually an {cls.name}")

class GenericAnimal(BaseAnimal):
    some_property: 100

class Elephant(BaseAnimal):
    name: str = "elephant"

generic_animal = GenericAnimal()
generic_animal.speak_static()
generic_animal.speak_class()

elephant = Elephant()
elephant.speak_static()
elephant.speak_class()

# Output:
# I am an animal
# I am actually an animal
# I am an animal
# I am actually an elephant
```

You can see that the speak_class uses the Elephant's name even though it doesn't know that the elephant's name beforehand.

This extra comes in handy when `Elephant` is defined in a whole other file, or even a whole other library, so you don't know it'll exist when you're defining the base.

You can also create a factory method using a class method which will build the derived class. You can combine this with introspection to make a very easily extensible factory method for a set of classes derived from a base class.

```py
class BaseAnimal():
    weight: int = 0
    pretty_name: str

    def __init__(self, pretty_name):
        self.pretty_name = pretty_name

    def speak(self):
        print(f"My name is {self.pretty_name},"
              f" I am an {type(self).__name__} and I weigh"
              f" {self.weight} kg.")
    
    @classmethod
    def try_make(cls, name, pretty_name):
        if cls.__name__ == name:
            return cls(pretty_name)
        else:
            return None

class Echidna(BaseAnimal):
    weight: int = 1

class Elephant(BaseAnimal):
    weight: int = 1000

Animals = [
    Echidna,
    Elephant
    # Add more here as necessary
]

def make_animal(name: str, pretty_name: str):
    for Animal in Animals:
        if instance := Animal.try_make(name, pretty_name):
            return instance

animal = make_animal("Elephant", "Jenny")
animal.speak()
# Output:
# My name is Jenny, I am an Elephant and I weigh 1000 kg.
```
