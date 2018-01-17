Some notes on design pattern, with examples in Python. I recommend implementing an example in your language for at least 4/5 design pattern

* The goal is to have a quick exposition on Design Pattern for potential questions
* Estimated Time: 5 hour

# What is a design pattern?
* It is a summary of a frequent design problem, plus the structure of a solution with its pros and cons, and finally a name so that it is easier to retain.
* Finding a good design pattern is hard. If we can use the design pattern for a certain problem, doesn't mean it is a good one.
* It is not a data structure, nor a system architecture, nor a programing language feature
* It must be studied in a language's context

## Creational Patterns
* Factory Method
```
def factory_method(param):
  if param == "smtg":
    return Object()
  elif param == "stmg else":
    return Object2()
  else:
    return Object3()
```

* Borg (is like Singleton): Not a single instance, but every instance has the same state
```
class Borg:
  _shared_state = {}
  def __init__(self):
    self.__dict__ = _shared_state
    self.state = "Init"
``` 

* Lazy initialization
* Builder: Instead of multiple constructors, the builder object receives parameters and returns constructed objects

## Structural Patterns
* 3-tier: To use for client/server applications: Split the application into 3:
  * D: data: class responsible of storing the data
  * B: business logic: class that can read/write the data
  * P: presentation: class that present the data
  B talk to D, and P talk to P
* Adapt the function call of another class for a certain class
```
class Adapter:
  def __init__(self, obj, **adapted_method):
    self.obj = obj
    self.__dict__.update(adapted_method)

  # if not in dict, it might be an obj method
  def __getattr__(attr):
    return getattr(self.obj, attr)

Adapter(cat, make_noise=cat.meow)
cat.make_noise()
```
* Composite: treating single instance, or composition of objects the same way
```
class A:
  def render():
    pass

class B:
  def render():
    pass

class CompositeObject():
  def __init__(self, objects):
    self.objects = objects
  def render():
    for object in self.objects:
      object.render()
```

* Observer
```
 # Test Observer pattern
 # Model is an subject that has observer
 from time import gmtime, strftime
 from abc import ABC, abstractmethod
 class Subject:
     def __init__(self):
         self.observers = []

     def add_observer(self, observer):
         self.observers.append(observer)

     def notify(self):
         for observer in self.observers:
             observer.update(self)

 class GetTime(Subject):
     def __init__(self):
         super().__init__()
         self._time = None

     def update_time(self):
         self._time = strftime("%H:%M:%S", gmtime())
         self.notify()

 class View:
     @abstractmethod
     def update(self, val):
         raise NotImplementedError()

 class ViewTime(View):
     def update(self, val):
         print(val._time)
```

* Delegate
```
class A:
  def do_something(): return a:
class B:
  def do_something(a): return a.do_something()
```

## Other design patterns

* RAII:  Ressource Acquisition Is Initialization
* Singleton: 
  A class has only one instance, and provide a global point of access to it. 
* Decorator
  Decorate an object
* Iterator
  Provide a way to access a sequential data without exposing its underlying structure
* Blockchain
* Borg pattern
  All instances are not the same identity as a Singleton, but they share the same state

```
class A:
    __shared_state = {}
    def __init__(self):
        self.__dict__ = self.__shared_state
```
