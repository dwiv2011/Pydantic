
🐍
Python Mastery Notes
Pydantic  ·  Decorators  ·  Validators  ·  Nested Models
✅ Runtime Validation	🔧 Type Coercion	🏗️ API Schemas

Comprehensive Study Reference  |  FastAPI & Production Python
 
1. Why Do We Need Pydantic?
Python is dynamically typed — it accepts almost anything at runtime, which can lead to silent bugs, inconsistent data, and brittle APIs. Pydantic solves this by enforcing types at runtime.

The Problem: No Enforcement
# Normal Python — accepts any input
def intro(name, age):
    print(name)
    print(age)

intro('Deepak', 35)       # ✅ Correct
intro('Deepak', 'abc')    # ❌ Wrong type — still runs!
intro('Deepak', 9999)     # ❌ Unrealistic — still runs!

⚠️  Problems Without Validation
❌  Data inconsistency — wrong types silently accepted
❌  Hidden bugs — errors appear far from the source
❌  Hard-to-debug systems — incorrect data travels deep
❌  Broken APIs — malformed requests pass through
❌  Unreliable ML pipelines — garbage in, garbage out

Partial Fix: Type Hints
Type hints document expected types but do NOT enforce them at runtime:
def intro2(name: str, age: int):  # Hint: age should be int
    print(name)
    print(age)

intro2('Deepak', 'abc')   # ❌ Still runs! Type hints are just hints.

💬  Type hints give IDE support and readability, but Python ignores them at runtime. They cannot replace real validation.

The Solution: Pydantic
🏆  What Pydantic Provides
✔  Runtime validation — rejects incorrect types immediately
✔  Automatic type coercion — converts '35' (string) → 35 (int) where possible
✔  Clear error messages — tells you exactly what went wrong
✔  Schema definition — describe the expected shape of your data
✔  JSON serialisation — convert models to dict / JSON easily

Used in: FastAPI  ·  Data Pipelines  ·  Configuration Management  ·  API Validation
 
2. Basic Pydantic Models
Defining a Model
Create a class that inherits from BaseModel. Each field is a class attribute with a type annotation:
from pydantic import BaseModel

class NewIntro(BaseModel):
    name: str
    age:  int

# ✅ Valid usage
newintro = NewIntro(name='Deepak', age=35)
print(newintro.name)   # Deepak
print(newintro.age)    # 35

# ✅ Create from a dictionary (common in APIs)
para = {'name': 'Deepak', 'age': 35}
input_value = NewIntro(**para)   # ** unpacks dict to keyword args

# ❌ Wrong type — Pydantic raises ValidationError
NewIntro(name='Deepak', age='abc')  # age must be int

The ** (Double Star) Operator
** unpacks a dictionary into keyword arguments — a common pattern when handling API request data:
With **  (unpacked)	Equivalent to
NewIntro(**para)	NewIntro(name='Deepak', age=35)

Automatic Type Coercion
Pydantic can automatically convert compatible types — this is called type coercion:
# Pydantic will try to convert the string '35' to int 35
user = NewIntro(name='Deepak', age='35')  # ✅ Works!

# But this fails — 'abc' cannot become an integer
user = NewIntro(name='Deepak', age='abc')  # ❌ ValidationError

🔄  Pydantic first tries to coerce. If coercion is impossible, it raises a ValidationError with a clear message telling you which field failed and why.

Complex Types: List and Dict
from pydantic import BaseModel
from typing import List, Dict

class NewIntro(BaseModel):
    name:            str
    age:             int
    hobby:           List[str]           # list of strings
    contact_details: Dict[str, str]      # key-value pairs

# ✅ Valid
para = {
    'name':            'Deepak',
    'age':             35,
    'hobby':           ['cricket', 'badminton', 'carrom'],
    'contact_details': {'phone': '1234', 'alt': '56789'}
}
user = NewIntro(**para)

# ❌ Missing field → ValidationError: contact_details field required
# ❌ Wrong key name 'Contact_details' → ValidationError
# ❌ List instead of Dict → ValidationError
 
3. Decorators in Python
Decorators are one of Python's most powerful features. A decorator is a function that takes another function, wraps it with extra behaviour, and returns the enhanced function.

Foundation: Functions Are Objects
In Python, functions are first-class objects — they can be stored, passed, and returned:
def greet():
    print('Hello')

a = greet     # Store function in a variable (no parentheses!)
a()           # Call it → prints 'Hello'

Functions Inside Functions (Closures)
def outer():
    def inner():
        print('Inside inner function')
    return inner     # Return the inner function

f = outer()          # f now holds the inner function
f()                  # → 'Inside inner function'

Building a Decorator Step by Step
Step 1 — Create a decorator function that wraps another function:
def my_decorator(func):          # Takes a function as argument
    def wrapper():               # Inner wrapper function
        print('Before call')
        func()                   # Call the original function
        print('After call')
    return wrapper               # Return wrapper (not the result!)

Step 2 — Apply the decorator manually:
def say_hello():
    print('Hello')

say_hello = my_decorator(say_hello)   # Wrap it manually
say_hello()                           # Call the wrapper

# Output:
# Before call
# Hello
# After call

Step 3 — Use the @ shorthand (syntactic sugar):
@my_decorator          # Exactly equivalent to: say_hello = my_decorator(say_hello)
def say_hello():
    print('Hello')

say_hello()            # Calls wrapper() which calls original say_hello()

🧠  Why We Need the wrapper() Function
Without wrapper, the decorator runs immediately when Python loads the function (not when you call it)
Without wrapper, my_decorator returns None, breaking the function
wrapper() stores the original function in a closure and runs it only when called
wrapper() delays execution until you actually call the decorated function
 
4. Decorators — Advanced Patterns
Universal Decorator with *args and **kwargs
The basic wrapper only works with functions that take no arguments. Use *args/**kwargs to support any function:
def my_decorator(func):
    def wrapper(*args, **kwargs):      # Accept any arguments
        print('Before execution')
        result = func(*args, **kwargs) # Pass arguments through
        print('After execution')
        return result                  # Return the result!
    return wrapper

@my_decorator
def add(a, b):
    return a + b

print(add(3, 5))
# Before execution
# After execution
# 8

Execution Flow Visualised
User calls  add(3, 5)
↓
Actually calls  wrapper(3, 5)
↓
print('Before')  →  func(3,5)  →  print('After')  →  return result

Real-World Use Cases
Use Case	Example
Logging	@log_execution — records every function call
Authentication	@login_required — blocks unauthenticated users
Timing / Profiling	@timer — measures execution time
Caching	@lru_cache — stores results of expensive calls
API Routing	@app.get('/users') — registers FastAPI endpoints
Validation	@field_validator('age') — validates Pydantic fields

Stacking Multiple Decorators
@decorator1
@decorator2
def func():
    pass

# Equivalent to:
func = decorator1(decorator2(func))

# Execution order: decorator2 wraps first, decorator1 wraps second
# When called: decorator1's wrapper runs first, then decorator2's

Class Decorators (Built-in)
Decorator	Effect
@classmethod	Method receives the class (cls) instead of an instance
@staticmethod	Method is independent — no cls or self
@property	Call a method like an attribute: obj.value not obj.value()
@field_validator	Registers a Pydantic field validation rule
@model_validator	Registers a Pydantic whole-model validation rule
@computed_field	Marks a property as a Pydantic model field
 
5. Field Validators in Pydantic
@field_validator lets you write custom validation logic for individual fields. It is triggered automatically when Pydantic processes that field.

Basic Structure
from pydantic import BaseModel, field_validator

class Patient(BaseModel):
    name:  str
    email: str
    age:   int

    @field_validator('email')       # Target field
    @classmethod                    # Always required
    def email_validator(cls, value):
        if '@' not in value:
            raise ValueError('Invalid email')
        return value                # Always return the value!

mode='before' vs mode='after'
The mode parameter controls when the validator runs in the pipeline:

mode="before"	mode="after"
Runs BEFORE type conversion

value is still the raw input

Use for: cleaning, stripping spaces, format conversion	Runs AFTER type conversion

value is already the target type

Use for: range checks, format validation (most common)

# mode='before' — value is raw string '  Deepak  '
@field_validator('name', mode='before')
@classmethod
def clean_name(cls, value):
    return value.strip()     # Clean before type check

# mode='after' — value is already int 35 (converted from '35')
@field_validator('age', mode='after')
@classmethod
def validate_age(cls, value):
    if value < 0 or value > 120:
        raise ValueError('Age must be between 0 and 120')
    return value

✅  Best Practice: Use mode='before' for cleaning raw input (strip, lowercase, reformat). Use mode='after' for logical validation (range, pattern, business rules).
 
6. Model Validators in Pydantic
@model_validator validates the entire model at once, giving access to all fields simultaneously. This is essential for cross-field validation rules.

When Field Validators Are Not Enough
🔗  Cross-Field Validation Examples
Healthcare: if age < 18 → married must be False
Banking: loan_amount must not exceed 10× salary
Insurance: premium_tier depends on both age and health_score
Employee systems: company email requires employee_id to exist

Model Validator with mode='after'
Runs after the model object is fully created — self gives access to all typed fields:
from pydantic import BaseModel, EmailStr, model_validator

class Patient(BaseModel):
    name:    str
    email:   EmailStr
    age:     int
    married: bool

    @model_validator(mode='after')      # Runs after type conversion
    def validate_patient(self):          # self = full model instance
        if self.age < 18 and self.married:
            raise ValueError('Patient under 18 cannot be married')
        return self                       # Always return self!

# ✅ Valid — 35-year-old can be married
p = Patient(name='Deepak', email='deepak@co.com', age=35, married=True)

# ❌ ValidationError — 15-year-old cannot be married
p = Patient(name='Raj', email='raj@co.com', age=15, married=True)

Model Validator with mode='before'
Runs before the model is created — cls receives a raw dictionary. Use this to pre-process or clean input:
@model_validator(mode='before')
@classmethod
def clean_data(cls, values):          # values = raw input dict
    if 'name' in values:
        values['name'] = values['name'].strip()
    return values                      # Return modified dict

Field Validator vs Model Validator
Feature	field_validator  vs  model_validator
Works on	Single field  ↔  Entire model
Access to other fields	Limited  ↔  Full access via self
Input received	Field value  ↔  Model instance or dict
Use case	Single-field rules  ↔  Cross-field rules
mode='before'	Raw field value  ↔  Raw input dict
mode='after'	Converted field value  ↔  Full model instance

Pydantic Validation Pipeline
1  Raw Input Data
2  model_validator(mode='before')
3  field_validator(mode='before')
4  Pydantic Type Conversion  ('35' → 35)
5  field_validator(mode='after')
6  model_validator(mode='after')
7  ✅ Final Model Object Created
 
7. Computed Fields & @property
computed_field lets you define fields that are calculated automatically from other fields — they are not part of the input data but are included in the model output.

Why @property Alone Is Not Enough
Approach	Result
@property only	Access works, but Pydantic ignores it in model output
@computed_field + @property	Pydantic includes it in dict/JSON output — correct approach

Full Example: Patient with BMI
from pydantic import BaseModel, EmailStr, computed_field
from typing import List, Dict

class Patient(BaseModel):
    name:            str
    email:           EmailStr
    age:             int
    weight:          float     # in kg
    height:          float     # in metres
    married:         bool
    allergies:       List[str]
    contact_details: Dict[str, str]

    @computed_field              # Tell Pydantic this is a model field
    @property                    # Access like attribute: patient.bmi
    def bmi(self) -> float:
        return round(self.weight / (self.height ** 2), 2)

patient_info = {
    'name':            'Deepak',
    'email':           'deepak@icici.com',
    'age':             '65',          # Will be coerced to int 65
    'weight':          75.2,
    'height':          1.72,
    'married':         True,
    'allergies':       ['pollen', 'dust'],
    'contact_details': {'phone': '2353462', 'emergency': '235236'}
}

patient1 = Patient(**patient_info)
print(patient1.bmi)   # 25.42  — accessed like attribute!

@property — The Key Insight
❌ Without @property
def bmi(self):
    return ...

patient.bmi()  # Must call it	✅ With @property
@property
def bmi(self):
    return ...

patient.bmi   # Natural access

Real-World Computed Field Examples
Domain	Computed Field
Healthcare	BMI from weight+height, risk score from age+conditions
Finance	interest_total from principal+rate+years
E-Commerce	total_price from unit_price×quantity, discount_price
HR Systems	years_of_service from hire_date to today
 
8. Nested Pydantic Models
Nested models allow you to embed one Pydantic model inside another, creating hierarchical data structures that mirror real-world relationships.

Defining Nested Models
from pydantic import BaseModel

class Address(BaseModel):    # Nested model
    city:  str
    state: str
    pin:   str

class Patient(BaseModel):    # Outer model
    name:    str
    gender:  str
    age:     int
    address: Address         # Nested model as a field type

Creating Nested Objects
# Step 1 — Create the nested object
address_dict = {'city': 'Bengaluru', 'state': 'Karnataka', 'pin': '560001'}
address1 = Address(**address_dict)

# Step 2 — Pass it into the outer model
patient_dict = {
    'name':    'Deepak',
    'gender':  'male',
    'age':     35,
    'address': address1        # Pydantic also accepts a plain dict here
}
patient1 = Patient(**patient_dict)

# Accessing nested data
print(patient1.address.city)   # Bengaluru

model_dump() — Convert to Dictionary
model_dump() serialises the entire model (including nested objects) to a Python dictionary:
patient1.model_dump()
# {
#   'name': 'Deepak', 'gender': 'male', 'age': 35,
#   'address': {'city': 'Bengaluru', 'state': 'Karnataka', 'pin': '560001'}
# }

# Include only specific fields
patient1.model_dump(include={'name', 'age'})
# {'name': 'Deepak', 'age': 35}

# Include specific nested fields
patient1.model_dump(include={'name', 'address': {'city'}})
# {'name': 'Deepak', 'address': {'city': 'Bengaluru'}}

# Exclude fields
patient1.model_dump(exclude={'gender'})

# Serialize directly to JSON string
patient1.model_dump_json()

model_dump() Parameter Reference
Parameter	Effect
model_dump()	Convert entire model to dict (including nested models)
include={'field1', 'field2'}	Only include the specified fields
include={'field': {'subfield'}}	Include specific sub-fields of nested model
exclude={'field'}	Exclude specified fields from output
model_dump_json()	Serialise directly to JSON string
 
9. Real-World: Pydantic in FastAPI
FastAPI uses Pydantic for everything — request validation, response serialisation, and API documentation. This is why mastering Pydantic is essential for FastAPI development.

A Complete FastAPI + Pydantic Example
from fastapi import FastAPI
from pydantic import BaseModel, EmailStr, field_validator, model_validator
from typing import Optional

app = FastAPI()

class UserCreate(BaseModel):        # Request body schema
    name:     str
    email:    EmailStr
    age:      int
    password: str
    confirm:  str

    @field_validator('name', mode='before')
    @classmethod
    def clean_name(cls, v):
        return v.strip().title()    # 'deepak' → 'Deepak'

    @field_validator('age')
    @classmethod
    def valid_age(cls, v):
        if v < 18:
            raise ValueError('Must be 18+')
        return v

    @model_validator(mode='after')
    def passwords_match(self):
        if self.password != self.confirm:
            raise ValueError('Passwords do not match')
        return self

@app.post('/create-user')           # Decorator registers route
def create_user(user: UserCreate):  # Pydantic validates automatically
    return {'message': f'User {user.name} created!'}

# FastAPI will:
# ✅ Parse JSON body into UserCreate
# ✅ Run all validators
# ✅ Return 422 error with details if validation fails
# ✅ Generate OpenAPI docs automatically

What FastAPI Does Automatically
🚀  FastAPI + Pydantic Integration Benefits
✔  Parses JSON request body into your Pydantic model
✔  Runs all field_validators and model_validators
✔  Returns HTTP 422 Unprocessable Entity with field-level error details
✔  Serialises response models back to JSON automatically
✔  Generates interactive Swagger/OpenAPI documentation
✔  Includes computed_field values in API responses
 
10. Quick Reference & Summary
Complete Concept Map
Concept	What It Does	Key Usage
BaseModel	Defines a validated data schema	Inherit to create any model
Type Hints	Declare expected field types	name: str, age: int
**kwargs	Unpack dict into model args	Model(**my_dict)
Type Coercion	Auto-convert compatible types	'35' → 35 automatically
field_validator	Validate a single field	@field_validator('age')
model_validator	Validate across all fields	@model_validator(mode='after')
mode='before'	Run before type conversion	Clean/strip raw input
mode='after'	Run after type conversion	Check ranges, business rules
computed_field	Auto-calculate derived field	@computed_field + @property
@property	Access method as attribute	patient.bmi not patient.bmi()
Nested Model	Embed model inside model	address: Address
model_dump()	Convert model to dict	For APIs, storage, JSON
EmailStr	Validate email format	email: EmailStr

The Golden Rule
Raw Input	→	Validate + Convert	→	✅ Safe Object

Validation Rule Cheat Sheet
Rule Type	Use This
Validate one field	@field_validator('field_name')
Validate multiple fields	@model_validator(mode='after')
Clean raw input first	mode='before' on field or model validator
Business logic on typed data	mode='after' on field or model validator
Calculate from other fields	@computed_field + @property
Represent nested data	Define a nested BaseModel class
Export to dict / JSON	.model_dump()  /  .model_dump_json()

📝  Remember: Pydantic does not just validate — it also documents. When you define a model, you are creating a self-describing schema that tools like FastAPI use to generate API documentation automatically.

