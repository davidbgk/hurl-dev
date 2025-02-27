---
layout: doc
title: Request
section: File Format
---
# {{ page.title }}

## Definition {#definition}

Describes an HTTP request, with mandatory [method](#method) and [url](#url), followed by optional [headers](#headers),
[query parameters](#query-parameters), [form parameters](#form-parameters), [multipart form datas](#multipart-form-data), 
[cookies](#cookies) and [body](#body).

## Example {#example}

```hurl
GET https://example.net/api/dogs?id=4567
User-Agent: My User Agent
Content-Type: application/json
```

## Description {#description}

### Method {#method}

Mandatory HTTP request method, one of `GET`, `HEAD`, `POST`, `PUT`, `DELETE`, `CONNECT`, `OPTIONS`,
`TRACE`, `PATCH`.

### URL {#url}

Mandatory HTTP request url.

Url can contain query parameters, even if using a [query parameters section](#query-parameters) is preferred.

```hurl
# A request with url containing query parameters.
GET https://example.net/forum/questions/?search=Install%20Linux&order=newest

# A request with query parameters section, equivalent to the first request.
GET https://example.net/forum/questions/
[QueryStringParams]
search: Install Linux
order: newest
```

> Query parameters in query parameter section are not url encoded.

When query parameters are present in the url and in a query parameters section, the resulting request will
have both parameters.

### Headers {#headers}

Optional list of HTTP request headers.

A header consists of a name, followed by a `:` and a value.

```hurl
GET https://example.net/news
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:70.0) Gecko/20100101 Firefox/70.0
Accept: */*
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

> Headers directly follow url, without any section name, contrary to query parameters, form parameters
> or cookies

Note that header usually don't start with double quotes. If the header value starts with double quotes, the double
quotes will be part of the header value:

```hurl
PATCH https://example.org/file.txt
If-Match: "e0023aa4e"
```

`If-Match` request header will be sent will the following value `"e0023aa4e"` (started and ended with double quotes).


### Query parameters {#query-parameters}

Optional list of query parameters.

A query parameter consists of a field, followed by a `:` and a value. The query parameters section starts with
 `[QueryStringParams]`. Contrary to query parameters in the url, each value in the query parameters section is not
 url encoded.

{% raw %}
```hurl
GET https://example.net/news
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:70.0) Gecko/20100101 Firefox/70.0
[QueryStringParams]
order: newest
search: {{custom-search}}
count: 100
```
{% endraw %}

If there are any parameters in the url, the resulted request will have both parameters.

### Form parameters {#form-parameters}

A form parameters section can be used to send data, like [HTML form](https://developer.mozilla.org/en-US/docs/Learn/Forms). 

This section contains an optional list of key values, each key followed by a `:` and a value. Key values will be 
encoded in key-value tuple separated by '&', with a '=' between the key and the value, and sent in the body request. 
The content type of the request is `application/x-www-form-urlencoded`. The form parameters section starts 
with `[FormParams]`.

{% raw %}
```hurl
POST https://example.net/contact
[FormParams]
default: false
token: {{token}}
email: john.doe@rookie.org
number: 33611223344
```
{% endraw %}

Form parameters section can be seen as syntactic sugar over body section (values in form parameters section
are not url encoded.). A [multiline string body](#multiline-string-body) could be used instead of a forms
parameters section.

~~~hurl
# Run a POST request with form parameters section:
POST https://example.net/test
[FormParams]
name: John Doe
key1: value1

# Run the same POST request with a body section:
POST https://example.net/test
Content-Type: application/x-www-form-urlencoded
```
name=John%20Doe&key1=value1
```
~~~

When both [body section](#body) and form parameters section are present, only the body section is taken into account.

### Multipart Form Data {#multipart-form-data}

A multipart form data section can be used to send data, with key / value and file content (see [multipart/form-data
 on MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/POST)). 
 
The form parameters section starts with `[MultipartFormData]`.

```hurl
POST https://example.net/upload
[MultipartFormData]
field1: value1
field2: file,example.txt;
# One can specify the file content type:
field3: file,example.zip; application/zip
```

Files are relative to the input Hurl file, and cannot contain implicit parent directory (`..`). You can use  
[`--file-root` option]({% link _docs/man-page.md %}#file-root) to specify the root directory of all file nodes.

Content type can be specified or inferred based on the filename extension:

- `.gif`: `image/gif`,
- `.jpg`: `image/jpeg`,
- `.jpeg`: `image/jpeg`,
- `.png`: `image/png`,
- `.svg`: `image/svg+xml`,
- `.txt`: `text/plain`,
- `.htm`: `text/html`,
- `.html`: `text/html`,
- `.pdf`: `application/pdf`,
- `.xml`: `application/xml`

By default, content type is `application/octet-stream`.


### Cookies {#cookies}

Optional list of session cookies for this request.

A cookie consists of a name, followed by a `:` and a value. Cookies are sent per request, and are not added to 
the cookie storage session, contrary to a cookie set in a header response. (for instance `Set-Cookie: theme=light`). The 
cookies section starts with `[Cookies]`.

```hurl
GET https://example.net/index.html
[Cookies]
theme: light
sessionToken: abc123
```

Cookies section can be seen as syntactic sugar over corresponding request header.

```hurl
# Run a GET request with cookies section:
GET https://example.net/index.html
[Cookies]
theme: light
sessionToken: abc123

# Run the same GET request with a header:
GET https://example.net/index.html
Cookie: theme=light; sessionToken=abc123
```

### Body {#body}

Optional HTTP body request. 

If the body of the request is a [JSON](https://www.json.org) string or a [XML](https://en.wikipedia.org/wiki/XML
) string, the value can be directly inserted without any modification. For a text based body that is not JSON nor XML
, one can use multiline string that starts with <code>&#96;&#96;&#96;</code> and ends with <code>&#96;&#96;&#96;</code>. 
For a precise byte control of the request body, [Base64](https://en.wikipedia.org/wiki/Base64) encoded string, 
[hexadecimal string](#hex-body) or [included file](#file-body) can be used to describe exactly the body byte content. 

> You can set a body request even with a `GET` body, even if this is not a common practice.

#### JSON body {#json-body}

JSON body is used to set a literal JSON as the request body.

```hurl
# Create a new doggy thing with JSON body:
POST https://example.net/api/dogs
{
    "id": 0,
    "name": "Frieda",
    "picture": "images/scottish-terrier.jpeg",
    "age": 3,
    "breed": "Scottish Terrier",
    "location": "Lisco, Alabama"
}
```

When using JSON body, the content type `application/json` is automatically set.

#### XML body {#xml-body}

XML body is used to set a literal XML as the request body.

~~~hurl
# Create a new soapy thing XML body:
POST https://example.net/InStock
Content-Type: application/soap+xml; charset=utf-8
Content-Length: 299
SOAPAction: "http://www.w3.org/2003/05/soap-envelope"
<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope" xmlns:m="http://www.example.org">
  <soap:Header></soap:Header>
  <soap:Body>
    <m:GetStockPrice>
      <m:StockName>GOOG</m:StockName>
    </m:GetStockPrice>
  </soap:Body>
</soap:Envelope>
~~~

#### Raw string body {#raw-string-body}

For text based body that are not JSON nor XML, one can used multiline string, started and ending with
<code>&#96;&#96;&#96;</code>.

~~~hurl
POST https://example.net/models
```
Year,Make,Model,Description,Price
1997,Ford,E350,"ac, abs, moon",3000.00
1999,Chevy,"Venture ""Extended Edition""","",4900.00
1999,Chevy,"Venture ""Extended Edition, Very Large""",,5000.00
1996,Jeep,Grand Cherokee,"MUST SELL! air, moon roof, loaded",4799.00
```
~~~

The standard usage of a raw string is :

~~~
```
line1
line2
line3
```
~~~

is evaluated as "line1\nline2\nline3\n".


To construct an empty string :

~~~
```
```
~~~

or

~~~
``````
~~~


Finaly, raw string can be used without any newline:

~~~
```line``` 
~~~

is evaluated as "line".


#### Base64 body {#base64-body}

Base64 body is used to set binary data as the request body.

Base64 body starts with `base64,` and end with `;`. MIME's Base64 encoding is supported (newlines and white spaces may be
 present anywhere but are to be ignored on decoding), and `=` padding characters might be added.

```hurl
POST https://example.net
# Some random comments before body
base64,TG9yZW0gaXBzdW0gZG9sb3Igc2l0IGFtZXQsIGNvbnNlY3RldHVyIG
FkaXBpc2NpbmcgZWxpdC4gSW4gbWFsZXN1YWRhLCBuaXNsIHZlbCBkaWN0dW0g
aGVuZHJlcml0LCBlc3QganVzdG8gYmliZW5kdW0gbWV0dXMsIG5lYyBydXRydW
0gdG9ydG9yIG1hc3NhIGlkIG1ldHVzLiA=;
```

#### Hex body {#hex-body}

Hex body is used to set binary data as the request body.

Hex body starts with `hex,` and end with `;`. 

```hurl
PUT https://example.net
# Send a café, encoded in UTF-8
hex,636166c3a90a;
```


#### File body {#file-body}

To use the binary content of a local file as the body request, file body can be used. File body starts with
`file,` and ends with `;``

```hurl
POST https://example.net
# Some random comments before body
file,data.bin;
```

File are relative to the input Hurl file, and cannot contain implicit parent directory (`..`). You can use  
[`--file-root` option]({% link _docs/man-page.md %}#file-root) to specify the root directory of all file nodes.
