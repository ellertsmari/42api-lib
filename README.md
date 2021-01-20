# HiveLib 42API (intra.py)

HiveLib 42API is a Python script that helps you connect and make requests to the 42 Network's internal [42API](https://api.intra.42.fr/apidoc).

## Pre-requisites:
First things first, get yourself **Python 3.6 or above**. You will also need the packages listed in the requirements.txt. Install them with the command `pip3 install -r requirements.txt`. We highly recommend using virtual environments for all Python projects, otherwise it might get [messy](https://xkcd.com/1987/). If you are new to Python, you may want to read some basics on [object oriented approach, classes and methods.](https://docs.python.org/3/tutorial/classes.html)

In order to use intra.py, you will also need a config.yml file. The YAML syntax is EZ, made to be human-readable and info about it can be found online. Then you need to take a look at the __init__ function in the intra.py, this will tell you what parameters you should put into the config.yml file.

Example config.yml file:
```yaml
intra:
  client: [REDACTED] # <- insert your app’s UID here
  secret: [REDACTED] # <- insert your app’s SECRET here
  uri: "https://api.intra.42.fr/v2/oauth/token"
  endpoint: "https://api.intra.42.fr/v2"
  scopes: "public"
```
For the client and secret parts, you will have to create an app, [read the manual](https://api.intra.42.fr/apidoc/guides/getting_started).


## Usage:
First, in your main .py file you need to import the `IntraAPIClient` class and create an instance of it:
```python
from intra import IntraAPIClient
ic = IntraAPIClient()
```

This instance will be your interface to do the requests with, as follows: 
```python
response = ic.get("http://api.intra.42.fr/v2/teams")
```

This example will `GET` you all the teams of all the campuses, returning a request object.

To work with the response data, you may want to convert it to a json object with the method .json():
```python
object = ic.get("http://api.intra.42.fr/v2/teams").json()
```

However this kind of request is not that useful. In fact, a single `.get()` request without any parameters only nets you the first 30 users, as the endpoint is paginated.

### Parameters:
If (should be when by now) you have read the API documentary, you may have noticed that you can apply all kinds of parameters to the request. These parameters include things like `sort`, `filter` and `range`. Make sure you always check the specific page in the documentation because different endpoints have different parameters and different ways of using them.

Parameters can be used to further specify your request without making the actual request string a mess. They are given as a parameter to the class method and should be in object format. An example of parameters and their usage:
```python
payload = {
   "filter[primary_campus]":13,
   "filter[cursus]":1,
   "range[final_mark]":"100,125",
   "sort":"-final_mark,name"
}
```

Here we are filtering by campus and cursus, results must be in a specified range of final_mark and they must be sorted in descending order based on final_mark and ascending order based on name.


To use the parameters with a certain request, you simply add them as a keyword argument params:
```python
response = ic.get("/projects/7/projects_users", params = payload)
```
Notice how with a correctly formatted YAML config, you can shorten the URL to only include the endpoint you need. The `IntraAPIClient` takes the given endpoint URL from config.yml file and appends to that whatever specific endpoint you are requesting.
```python
response = ic.get("teams", params = payload)
data = response.json()
```

### Pagination:
Most of the endpoints are paginated with the request parameters `page` and `per_page`. In order to receive all of the data of a certain endpoint, you usually need to do multiple requests. The `ic.pages()` method gets you all the data of a specified endpoint. It will check the total amount of data available and keep requesting pages until it has pooled all the data from the response. It returns the data as a list of  .json() objects instead of the regular single response object. With `.pages()` you should take extra care to provide appropriate parameters, in order to avoid doing hundreds of requests and taking excessive amounts of time. Most of the time you can avoid getting a huge amount of pages with appropriate parameter usage. You can also check the response header `X-Total` to get the total count of items on the response. `ic.pages_threaded()` does the same thing as `ic.pages()`, but in multiple threads, thus reducing the time it takes to retrieve bigger requests to about half.

Here we get all of the users for all of the 42 campuses ever using 20 threads and end up using around 700 requests to do it:
```python
userList = ic.pages_threaded("/users")
```

## Spotted an error? Wanna add something?
Why not fix it and make a pull request! You just might end up getting a small reward for it...
