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





