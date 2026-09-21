# 1. First Steps
## Create the simplest possible FastAPI file

```python
from fastapi import FastAPI # Import FastAPI

app = FastAPI() # Create an instance of FastAPI and assign it to a variable called 'app'

@app.get("/") # Use the @app.get("/") decorator to create a path operation for the root path
async def root(): # Define an async function 
    return {"message": "Hello World"} 
```

# 2. Path Parameters
## Learn to create dynamic URLs that capture values from the URL path

```python
from fastapi import FastAPI

app = FastAPI()

# Accept item_id as a parameter (no type hint initially)
@app.get("/items/{item_id}") 
async def get_item_id(item_id):
    return {"item_id": item_id}

# Accept item_id as an integer (use type hint: item_id: int)
# This demonstrates automatic type conversion!
@app.get("/items/{item_id}/typed")
async def get_typed(item_id: int):
    return  {"item_id": item_id}

# Take no parameters
# This must come BEFORE the variable path!
@app.get("/users/me")
async def get_me():
    return {"user_id": "the current user"}
# Accept user_id as a string parameter
@app.get("/users/{user_id}")
async def get_user(user_id):
    return {"user_id": user_id}
```

# 3. Query Parameters - Following the Official FastAPI Tutorial
## Learn how to handle optional parameters in URLs

```python
from fastapi import FastAPI

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

# Return the slice fake_items_db[skip : skip + limit]
# Hint: Function parameters become query parameters automatically!
@app.get("/items/")
async def query_para(skip: int = 0, limit: int = 10):
    return fake_items_db[skip : skip+limit] 

# Return {"item_id": item_id, "q": q} when q was given, and {"item_id": item_id} when it was not
# Mix path parameters {item_id} with query parameters q
@app.get("/items/{item_id}")
async def query_para_q(item_id, q: str | None = None):
    if q:
        return {"item_id": item_id, "q": q} 
    return {"item_id": item_id} 
```

# 4.Request Body 

```python
from fastapi import FastAPI
from pydantic import BaseModel

app = FastAPI()
# Create a Pydantic model
class Item(BaseModel): # Create a class called 'Item' that inherits from BaseModel
    name: str
    description: str | None = None
    price: float
    tax: float | None = None

@app.post("/items/")
def creat_item(item: Item):
    return item


@app.post("/items/{item_id}")
def creat_item_with_id(item_id: int, item: Item):
    return  {"item_id": item_id, **item.model_dump()} # {"item_id": item_id, **item.dict()}
```
**item.model_dump() Turn:
{
  "item_id": 1,
  "item": {
    "name": "Phone",
    "description": null,
    "price": 999.0,
    "tax": null
  }
} 
to:
{
  "item_id": 1,
  "name": "Phone",
  "description": null,
  "price": 999.0,
  "tax": null
}

# 5. Query Parameters and String Validations 

```python
from fastapi import FastAPI
from fastapi import Query

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

# Return items filtered by q if provided, or all items if q is None
@app.get("/items/")
def para(q: str | None = Query(default=None, max_length=50)): # Use Query() instead of just setting default values
    result = []
    if q:
        results = [item for item in fake_items_db if q.lower() in item["item_name"].lower()]
        return results
    else:
        return fake_items_db

# Return: {"query": q, "limit": limit, "results": [...]}

@app.get("/items/search/")
def para_more(
    q: str = Query(min_length=3, max_length=50, description="Search query"), 
    skip: int = Query(default=0, ge=0, description="Skip offset"), # ge=1 means "greater than or equal to 1"
    limit: int = Query(10, ge=1, le=100, description="Maximum number of items")):

    results = []
    if q:
        results = [item for item in  fake_items_db if q.lower() in item["item_name"].lower()]
        results = results[:limit]
    else:
        results = fake_items_db[:limit]
    return results[skip:skip+limit]
```

# 6. Path Parameters and Numeric Validations

```python
from fastapi import FastAPI
from fastapi import Path, Query

app = FastAPI()

fake_items_db = [{"item_name": "Foo"}, {"item_name": "Bar"}, {"item_name": "Baz"}]

# Add validation to path parameters
@app.get("/items/{item_id}")
def read_item(
    item_id: int = Path(ge=1)
    ):
    return  {"item_id": item_id}

# Combine Path and Query validations
@app.get("/items/{item_id}/details")
def get_detail(
    item_id: int = Path(ge=1, le=1000, description="The ID of the item"),
    q: str | None = Query(default=None, max_length=50)):
    return {"item_id": item_id, "q": q, "details": "Item details here"}

# Add metadata to path parameters
# f-string: {"message": f"User {user_id} profile"}
@app.get("/users/{user_id}")
def get_user_id(
    user_id: int = Path(title="User ID", description="The ID of the user to get", ge=1)):
    return {"user_id": user_id, "message": f"User {user_id} profile"}
```

# Body - Multiple Parameters

```python
from typing import Union
from fastapi import FastAPI, Path, Body
from pydantic import BaseModel

app = FastAPI()

# Pydantic models for request bodies
class Item(BaseModel):
    name: str
    description: Union[str, None] = None
    price: float
    tax: Union[float, None] = None

class User(BaseModel):
    username: str
    full_name: Union[str, None] = None

# Mix Path, Query and body parameters
# Return results dict with item_id, and conditionally add q and item if provided
@app.put("/items/{item_id}/basic")
def mix_put(
    item_id: int = Path(title="The ID of the item to get", ge=0, le=1000), 
    q: str | None = None,
    item: Item | None = None):
    results =  {"item_id": item_id}
    if q:
        results["q"] = q
    if item:
        results["item"] = item
    return results

# Multiple body parameters
@app.put("/items/{item_id}")
def put_para(
    item_id: int,
    item: Item,
    user: User
    ):
    return  {"item_id": item_id, "item": item, "user": user}

# Singular values in body
# Return: dict with all parameters
@app.put("/items/{item_id}/importance")
def put_sin(
    item_id: int, item: Item, user: User, 
    importance: int = Body()
    ):
    return {"item_id": item_id, "item": item, "user": user, "importance": importance}
    
# Multiple body params and query
# Use * to force keyword-only arguments
# Return: dict with all params, conditionally add q
@app.put("/items/{item_id}/full")
def put_body(
    *, # '*' allows 'q=None' to sit before non-default params without SyntaxError
    item_id: int, item: Item, user: User, 
     importance: int = Body(gt=0),
     q: str | None = None):
     results = {"item_id": item_id, "item": item, "user": user, "importance": importance}
     if q:
        results["q"] = q
     return results

# Embed single body parameter
@app.put("/items/{item_id}/embed")
def put_embed(
    item_id: int, item: Item = Body(embed=True) ):
    return {"item_id": item_id, "item": item}
```



    







