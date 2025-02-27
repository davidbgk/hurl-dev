---
layout: doc
title: Chaining Requests
section: Tutorial
---
# {{ page.title }}

## Adding Another Request

Our basic Hurl file is for the moment:

```hurl
# Checking our home page:
GET http://localhost:8080
HTTP/1.1 200
[Asserts]
xpath "string(//head/title)" == "Welcome to Quiz!"
xpath "//button" count == 2
xpath "string((//button)[1])" contains "Play"
xpath "string((//button)[2])" contains "Create"
# Testing content type:
header "Content-Type" == "text/html;charset=UTF-8"
# Testing session cookie:
cookie "JSESSIONID" exists
cookie "JSESSIONID[HttpOnly]" exists
```

We're only running one HTTP request and have already added lots of tests on the response. Don't hesitate to add
many tests, the more asserts you will write, the less fragile will be your tests suite.

Now, we want to perform other HTTP requests and keep adding tests. In the same file, we can simply write another 
request following our first request. Let's say we want to test that we have a [404 page] on a broken link:

1. Modify `basic.hurl` to add a second request on a broken url:

```hurl
# Checking our home page:
GET http://localhost:8080
HTTP/1.1 200
[Asserts]
xpath "string(//head/title)" == "Welcome to Quiz!"
xpath "//button" count == 2
xpath "string((//button)[1])" contains "Play"
xpath "string((//button)[2])" contains "Create"
# Testing content type:
header "Content-Type" == "text/html;charset=UTF-8"
# Testing session cookie:
cookie "JSESSIONID" exists
cookie "JSESSIONID[HttpOnly]" exists

# Check that we have a 404 response for broken links:
GET http://localhost:8080/not-found
HTTP/1.1 404
[Asserts]
header "Content-Type" == "text/html;charset=UTF-8"
xpath "string(//h1)" == "Error 404, Page not Found!"
```

Now, we have two entries in our Hurl file: each entry is composed of one request and one expected response 
description. 

> In a Hurl file, response description are optional. We could also have written 
> our file with only requests:
>
> ```hurl
> GET http://localhost:8080
> GET http://localhost:8080/not-found
> ```
> But it would have performed nearly zero test. This type of Hurl file can be useful
> if you use Hurl to get data for instance.

{:start="2"}
2. Run `basic.hurl`:

```
$ hurl basic.hurl
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title></title>
    <link rel="stylesheet" href="/style.css">
    <!--<script src="script.js"></script>-->
</head>
<body>
<div>
    <img class="logo" src="/quiz.svg" alt="Quiz logo">
</div>
<h1>Error 404, Page not Found!</h1>

<a href="/">Quiz Home</a>


</body>
</html>
```

We can see that the test is still ok, but now, Hurl outputs the response of the last HTTP request (i.e. 
the content of our 404 page). This is useful when you want to get data from a server, and you need to 
perform additional steps (like login, confirmation etc...) before being able to call your last request. 

In our tutorial, we're simply interested to verify the success or failure of our integration tests.
So, first, we'll remove the standard output (if a test is broken, we'll still have the error output).

{:start="3"}
3. Run `basic.hurl` while redirecting the standard ouput to `/dev/null`:

```
$ hurl basic.hurl > /dev/null
```

Then, we can also use [`--progress`] and [`--summary`] option to give us some feedback on 
our tests progression and a simple summary:

{:start="4"}
4. Run `basic.hurl` with `--progress` and `--summary` options:

```
$ hurl --progress --summary basic.hurl > /dev/null
basic.hurl: RUNNING [1/1]
basic.hurl: SUCCESS
--------------------------------------------------------------------------------
Executed:  1
Succeeded: 1 (100.0%)
Failed:    0 (0.0%)
Duration:  40ms
```

Finally, we can use the [`--test`] option that is a shortcut for redirecting the standard output, 
, using [`--progress`] and [`--summary`] options:

{:start="5"}
5. Run `basic.hurl` with `--test` option:

```
$ hurl --test basic.hurl
basic.hurl: RUNNING [1/1]
basic.hurl: SUCCESS
--------------------------------------------------------------------------------
Executed:  1
Succeeded: 1 (100.0%)
Failed:    0 (0.0%)
Duration:  40ms
```

From now on, we will always use `--test` to run our tests files.

## Test REST Api

So far, we have tested two HTML endpoints. We're going to see now how to test a REST api. 

Our quiz application exposes a health REST resource, available at <http://localhost:8080/api/health>.
Let's use Hurl to check it.

1. In a shell, use Hurl to test the </api/health> endpoint:

```
$ echo 'GET http://localhost:8080/api/health' | hurl
{"status":"RUNNING","reportedDate":"2021-06-06T14:08:27Z","healthy":true,"operationId":425276758}
```

> Being a classic CLI application, we can use the standard input with Hurl to provide requests
> to be executed, instead of a file.

So, our health api returns this JSON resource:

```
{
  "status": "RUNNING",
  "reportedDate": "2021-06-06T14:08:27Z",
  "healthy": true,
  "operationId": 425276758
}
```


We can test it with a [JsonPath assert]. JsonPath asserts have the same structure as XPath asserts: a query 
followed by a predicate. [JsonPath query] are simple expressions to inspect a JSON object. 

{:start="2"}
2. Modify `basic.hurl` to add a third request that asserts our </api/health> REST api:

```hurl
# Checking our home page:
# ...

# Check that we have a 404 response for broken links:
# ...

# Check our health api:
GET http://localhost:8080/api/health
HTTP/1.1 200
[Asserts]
header "Content-Type" == "application/json"
jsonpath "$.status" == "RUNNING"
jsonpath "$.healthy" == true
jsonpath "$.operationId" exists
```

Like XPath assert, JsonPath predicates values are typed. String, boolean, number and
collections are supported. Let's practice it by using another api. In our Quiz model, a 
quiz is a set of questions, and a question resource is exposed through a 
REST api exposed et <http://localhost:8080/api/questions>. We can use it to add checks on getting questions 
through the api endpoint.

{:start="3"}
3. Add JSONPath asserts on the </api/questions> REST apis:

```hurl
# Checking our home page:
# ...

# Check that we have a 404 response for broken links:
# ...

# Check our health api:
# ...

# Check question api:
GET http://localhost:8080/api/questions?offset=0&size=20&sort=oldest
HTTP/1.1 200
[Asserts]
header "Content-Type" == "application/json"
jsonpath "$" count == 20
jsonpath "$[0].id" == "c0d80047"
jsonpath "$[0].title" == "What is a pennyroyal?"
```

> To keep things simple in this tutorial, we have hardcoded mocked data
> in our Quiz application. That's something you don't want to do when building 
> your application, you want to build an app production ready. A better way to 
> do this should have been to expose a "debug" or "integration" mode on our app
> positioned by environnement variables. If our app is launched in "integration" mode,
> mocked data is used and asserts can be tested on known values. Our app could also use
> a mocked database, configured in our tests suits.

Note that the question api use query parameters `offset`, `size` and `sort`, that's why we have written the url with 
query parameters <http://localhost:8080/api/questions?offset=0&size=20&sort=oldest>. We can set the query parameters 
in the url, or use a [query parameter section].

{:start="4"}
4. Use a query parameter section in `basic.hurl`:

```hurl
# Checking our home page:
# ...

# Check that we have a 404 response for broken links:
# ...

# Check our health api:
# ...

# Check question api:
GET http://localhost:8080/api/questions
[QueryStringParams]
offset: 0
size: 20
sort: oldest
HTTP/1.1 200
[Asserts]
header "Content-Type" == "application/json"
jsonpath "$" count == 20
jsonpath "$[0].id" == "c0d80047"
jsonpath "$[0].title" == "What is a pennyroyal?"
```

Finally, our basic Hurl file, with four requests, looks like:

```hurl
# Checking our home page:
GET http://localhost:8080
HTTP/1.1 200
[Asserts]
xpath "string(//head/title)" == "Welcome to Quiz!"
xpath "//button" count == 2
xpath "string((//button)[1])" contains "Play"
xpath "string((//button)[2])" contains "Create"
# Testing content type:
header "Content-Type" == "text/html;charset=UTF-8"
# Testing session cookie:
cookie "JSESSIONID" exists
cookie "JSESSIONID[HttpOnly]" exists

# Check that we have a 404 response for broken links:
GET http://localhost:8080/not-found
HTTP/1.1 404
[Asserts]
header "Content-Type" == "text/html;charset=UTF-8"
xpath "string(//h1)" == "Error 404, Page not Found!"

# Check our health api:
GET http://localhost:8080/api/health
HTTP/1.1 200
[Asserts]
header "Content-Type" == "application/json"
jsonpath "$.status" == "RUNNING"
jsonpath "$.healthy" == true
jsonpath "$.operationId" exists

# Check question api:
GET http://localhost:8080/api/questions
[QueryStringParams]
offset: 0
size: 20
sort: oldest
HTTP/1.1 200
[Asserts]
header "Content-Type" == "application/json"
jsonpath "$" count == 20
jsonpath "$[0].id" == "c0d80047"
jsonpath "$[0].title" == "What is a pennyroyal?"
```

{:start="5"}
5. Run `basic.hurl` and check that every assert of every request has been successful:

```
$ hurl --test basic.hurl
basic.hurl: RUNNING [1/1]
basic.hurl: SUCCESS
--------------------------------------------------------------------------------
Executed:  1
Succeeded: 1 (100.0%)
Failed:    0 (0.0%)
Duration:  33ms
```

## Recap

We can simply chain requests with Hurl, adding asserts on every response. As your Hurl file will grow, 
don't hesitate to add many comments: your Hurl file will be a valuable and testable documentation 
for your applications.

[404 page]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/404
[JsonPath assert]: {% link _docs/asserting-response.md %}#jsonpath-assert
[JsonPath query]: https://goessner.net/articles/JsonPath/
[query parameter section]: {% link _docs/request.md %}#query-parameters
[`--progress`]: {% link _docs/man-page.md %}#progress
[`--summary`]: {% link _docs/man-page.md %}#summary
[`--test`]: {% link _docs/man-page.md %}#test
