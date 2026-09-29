# Object-Oriented Programming (OOP) Essentials: Complete Guide

A complete, beginner-friendly reference covering classes, objects, attributes, methods, constructors, subclassing, and composition in Python.

---

## 1. Class vs. Object

In Object-Oriented Programming (OOP), everything revolves around **Classes** and **Objects**.

* **Class (The Blueprint / Template):** A class defines what data an entity will hold and what actions it can perform. It is not a live thing in memory yet; it is just a pattern.
* **Object (The Instance):** An object is the actual concrete thing created from that blueprint. It holds real data and lives in memory.

### Real-World Analogy

* **Class:** Architectural blueprint for a house.
* **Object:** The actual physical house built on a lot using that blueprint.

---

## 2. Constructor (`__init__`)

A **constructor** is a special function (method) inside a class that automatically runs the moment a new object is created from that class.

* In Python, it is defined as `def __init__(self, ...):`.
* Its main job is to **initialize state** (assign initial variables/attributes to the object).
* `self` refers to the specific instance (object) currently being created.

---

## 3. Attributes vs. Methods

Inside an object, functionality and data are divided into two categories:

| Feature | What it represents | OOP Term | Parentheses `()` needed? | Example |
| --- | --- | --- | --- | --- |
| **Attribute** | What an object **has / is** (Data / Variable) | Property / Field | **No** | `person.name` |
| **Method** | What an object **does** (Action / Function) | Behavior / Method | **Yes** | `person.say_hello()` |

### Real-World Analogy (A Car)

* **Attributes:** `car.color = "Red"`, `car.speed = 60`
* **Methods:** `car.accelerate()`, `car.brake()`

---

## 4. Basic Python Example (Class, Object, Constructor, Attributes, Methods)

```python
class Person:
    # Constructor: Runs automatically when creating a new object
    def __init__(self, name: str, age: int):
        # ATTRIBUTES (Data stored inside the object)
        self.name = name
        self.age = age

    # METHOD (Action inside the object)
    def say_hello(self):
        print(f"Hello, my name is {self.name} and I am {self.age} years old.")


# --- CREATING OBJECTS (Instantiation) ---
person1 = Person("Alice", 25)  # Instantiates Person object
person2 = Person("Bob", 30)    # Instantiates another Person object

# Accessing Attributes (No parentheses)
print(person1.name)  # Output: Alice

# Calling Methods (Requires parentheses)
person1.say_hello()  # Output: Hello, my name is Alice and I am 25 years old.

```

---

## 5. Composition (Attributes holding Sub-Objects)

An attribute does not have to be a simple primitive (like a `string` or `number`). An attribute can hold an **entire object** created from another class.

This is called **Composition** (a *"Has-A"* relationship).

### Real-World Analogy (Smartphone & Camera)

A smartphone is not a camera, but a smartphone **has a** camera inside it.

* **`phone`** $\rightarrow$ Main Object
* **`.camera`** $\rightarrow$ Attribute holding the nested camera tool object
* **`.take_photo()`** $\rightarrow$ Method inside the camera tool object

**Code structure:** `phone.camera.take_photo()`

### Python Code Example (Composition)

```python
class Camera:
    """A helper class representing a camera tool."""
    def take_photo(self):
        print("📸 Photo captured!")


class Phone:
    """Main class that contains helper objects as attributes."""
    def __init__(self, brand: str):
        self.brand = brand
        
        # COMPOSITION: The 'camera' attribute holds an instance of the Camera class
        self.camera = Camera()


# Usage
my_phone = Phone("Apple")

# 1. Access 'camera' attribute on 'my_phone' -> gets Camera instance
# 2. Call '.take_photo()' method on that Camera instance
my_phone.camera.take_photo()  # Output: 📸 Photo captured!

```

---

## 6. Real-World SDK Example: How `openai` Works

Libraries like the `openai` Python SDK use **Composition** to structure their code cleanly using dot notation (`client.responses.create(...)`).

### Why use Composition in an SDK?

If all API methods were placed on the main `client` object, it would become cluttered with flat methods (`client.create_response()`, `client.upload_file()`, `client.list_models()`).

Grouping related features into **resource attributes** creates clean namespaces:

* `client.responses.create(...)`
* `client.files.upload(...)`
* `client.models.list(...)`

### Simulated SDK Source Code

Here is a simplified Python source code example showing how a SDK library like `openai` is structured under the hood.

---

### Simplified Source Code Structure

#### 1. The Helper Resource Class (`responses.py`)

This class defines the manager object that holds specific methods like `create()`.

```python
# openai/resources/responses.py

class Responses:
    def __init__(self, client):
        # Stores a reference back to the main client (for API keys, config, etc.)
        self._client = client

    def create(self, model: str, input: str):
        """Method to trigger the API network call for a response."""
        api_key = self._client.api_key
        print(f"Sending API request with key '{api_key}'...")
        print(f"Model: {model} | Input: '{input}'")
        
        # Simulating returning a response object from the API
        return {"output_text": "Here is the model output."}

```

---

#### 2. The Main Client Class (`client.py`)

When you initialize `OpenAI()`, its `__init__` constructor creates instance variables (attributes) and assigns new helper objects to them.

```python
# openai/client.py
from openai.resources.responses import Responses

class OpenAI:
    def __init__(self, api_key: str = "default_sk_key"):
        self.api_key = api_key

        # --- ATTRIBUTES (Holding Helper Objects) ---
        # Here, the attribute 'responses' is initialized as an object instance of Responses
        self.responses = Responses(client=self)
        
        # Other resource attributes would be set up the same way:
        # self.files = Files(client=self)
        # self.models = Models(client=self)

```

---

#### 3. Package Entry Point (`__init__.py`)

This exposes the `OpenAI` class directly at the top level of the `openai` package.

```python
# openai/__init__.py
from openai.client import OpenAI

__all__ = ["OpenAI"]

```

---

### How Everything Connects in Your Code

When you write:

```python
from openai import OpenAI

# 1. Instantiates OpenAI class -> executes OpenAI.__init__()
# 2. Inside __init__, self.responses is assigned an instance of Responses
client = OpenAI()

# 3. Access 'responses' attribute -> retrieves the Responses object instance
# 4. Calls '.create()' method on that object instance
response = client.responses.create(
    model="gpt-6-astra",
    input="Hello world"
)

```

**Key Takeaway:** The attribute (`client.responses`) points to a **class instance** (`Responses`), and the method (`.create()`) is a standard function inside that class.

---

## 7. Subclassing (Inheritance) vs. Composition (Attributes)

| Concept | OOP Principle | Relationship | What it means | Source Code Example | Usage Example |
| --- | --- | --- | --- | --- | --- |
| **Subclassing** | Inheritance | **Is-A** | Class B is a specialized version of Class A | `class ElectricCar(Car):` | `car = ElectricCar()`<br>

<br>`car.charge()` |
| **Composition** | Attributes | **Has-A** | Class A contains an instance of Class B as a tool | `self.engine = Engine()` | `car = Car()`<br>

<br>`car.engine.start()` |

---

### Why Subclassing isn't used for API Resources (Like `client.responses`)

#### 1. Conceptual Mismatch ("Is-A" vs. "Has-A")

* **Subclassing:** Saying `class Responses(OpenAI):` implies a Response manager **is an** OpenAI client. This is logically incorrect.
* **Composition:** Saying `self.responses = ResponsesManager(self)` implies an OpenAI client **has a** Response manager tool. This correctly reflects the architecture.

#### 2. Technical Limitation (Passing Shared State)

In Python, subclasses inherit class structure, but **do not share instance data** (like runtime configuration or API keys) automatically.

If subclassing were used, you would have to pass your API key repeatedly every time you instantiated a new feature tool:

```python
# ❌ HYPOTHETICAL BAD DESIGN (Using Subclassing for features):
class Responses(OpenAI):  # Inherits from OpenAI
    def create(self, model, input):
        print(f"Using key: {self.api_key}")

# User must re-pass the API key for every single tool instance:
responses_tool = Responses(api_key="sk-12345")
files_tool = Files(api_key="sk-12345")
models_tool = Models(api_key="sk-12345")

```

With **Composition (Attributes)**, you pass configuration **once** to the main `client`, and all sub-tool attributes access that single central client instance automatically:

```python
# ✅ GOOD DESIGN (Using Composition / Attributes):
client = OpenAI(api_key="sk-12345")

# All tools read from 'client' automatically:
client.responses.create(...)
client.files.upload(...)
client.models.list(...)

```
