
# Distributed Systems: Web Services
+ **Felix Garcia Carballeira and Alejandro Calderon Mateos**
+ [![License: CC BY-NC-SA 4.0](https://img.shields.io/badge/License-CC%20BY--NC--SA%204.0-blue.svg)](https://github.com/acaldero/uc3m_sd/blob/main/LICENSE)


## Contents

  * [Motivation for using Web Services](#introduction)
  * [Service Styles: SOAP vs RESTful](#service-styles-soap-vs-rest)
  * REST and SOAP examples:
    * [Python - REST: Simple example of a web service  (server and client)](#simple-example-of-a-rest-web-service-server-and-client-in-python)
    * [Python - REST: Simple example with OpenAPI (server and client)](#openapi)  
    * [Python - SOAP: Web service example  (client)](#simple-web-service-example-soap-client-in-python)
    * [Python - SOAP: Simple web service example (client and server) ](#simple-example-of-soap-web-service-client-and-server-in-python)
    * [C - SOAP: Using a distributed service based on gSOAP/XML (client only)](#using-a-distributed-service-based-on-gsoapxml-client-only-in-c)
    * [C - SOAP: Creating a distributed service based on gSOAP/XML (client and server)](#creating-a -distributed-service-based-on-gsoapxml-client-and-server-in-c)
  * [Other technologies besides REST and SOAP](#other-technologies-besides-rest-and-soap)
  * Examples of other technologies:
    * [Simple example of a web service based on server-sent events (SSE)](#simple-example-of-a-web-service-based-on-sse-in-bash)
    * [Creating a distributed service based on Apache Thrift (client and server, in Python)](#creating-a-distributed-service-based-on-apache-thrift-client-and-server-in-python)
    * [Creating a distributed service based on gRPC (client and server, in Python)](#creating-a-distributed-service-based-on-grcp-client-and-server-in-python) -based-on-grpc-client-and-server-in-python)
    * [Example of JSON-RPC over HTTP: simple calculator MCP server (client and server, in Python)](#example-of-json-rpc-over-http-simple-calculator-mcp-server-client-and-server-in-python)


## Introduction

Web services are at the level of a network service and one level above the remote procedure paradigm:

   ![Paradigms by level](materials/topic-ws/web-services/ssdd_web-services-drawio_4.svg)
                          
Web services (and network services) are an extension to the remote procedure invocation paradigm in which a directory service is added that provides references to available services:

   ![Location transparency](/materials/topic-ws/web-services/ssdd_web-services-drawio_5.svg)

This directory service allows for *location transparency*, which is an extra level of abstraction.


#### Location transparency

The usual steps for using the directory service (or registry) are:
1. The requesting process contacts the directory service and searches for a service
2. The directory service returns the reference to the requested service
3. Using the reference, the requesting process interacts with the service

The main differences with RPC are:
   * In an RPC, the port of the machine where the *“directory service”* is running is searched for
     * In Web services, the list of URLs (protocol, machine, and port) where the service is located is searched
   * In an RPC, a service identifier number is searched
     * In Web services, you can search by name, service characteristics, etc.


#### Remote invocation using the Web

In the directory, you can register the service (step 0) and deregister it.
Registering a service may involve including the following in the directory:
* Service identification.
* Address and port of the machine where it is offered.
* A description of the API that allows you to design, implement, and test a client (consumer) that requests the service from a server (provider).

From the API description, it may be possible to generate stubs that facilitate deployment in a similar way to how `rpcgen` automates the generation of part of the code in RPCs.
* Generating code using a tool of this type ensures that the generation process has been validated (rather than relying on stochastic code generation that may not work, as is the case with purely AI-based tools).


#### What has facilitated web services

Having a repository of web services available allows:
* **Different software applications** developed in different programming languages and running on any platform can use web services to exchange data.
* A single **distributed application** can be broken down into independent, loosely coupled modular services that can interoperate with each other.
     * * *SOA architecture**: Architecture in which software is exposed as a “service” that is invoked using a standard communication protocol
     *  It is possible to compose and combine web services to generate new distributed applications (e.g., a travel agency that interacts with hotels, airlines, banks, etc.).


## First generation of the WWW: static content

The WWW uses a client (e.g., web browser) and server (e.g., Apache web server) architecture, with a text-based HTTP protocol. \
For example, it is possible to request a web page manually:
* We execute:
  ```
  $ telnet www.lab.inf.uc3m.es 80
  ```
  And the output is:
  ```
  Trying 163.117.61.51...
  Connected to waf.uc3m.es.
  Escape character is '^]'.
  ```
* We type:
  ```
  GET / HTTP/1.0
  accept: */*
  ```
  And the output is:
  ```html
  HTTP/1.1 301 Moved Permanently
  Date: Fri, 28 Apr 2023 11: 18:58 GMT
  Server: Apache
  Location: https:///
  Content-Length: 217
  Connection: close
  Content-Type: text/html; charset=iso-8859-1

  <!DOCTYPE HTML PUBLIC "-//IETF// DTD HTML 2.0//EN">
  <html><head>
  <title>301 Moved Permanently</title>
  </head><body>
  <h1>Moved Permanently</h1>
  <p>The document has moved <a href="https:///">here</a>.</p>
  </body></html>
  Connection closed by foreign host.
  ```

The output has two parts:
* A header with the response metadata.
* The content of the requested page (index.html) in HTML text format.

Initially, web servers were used to send static content stored in files. \
This first generation allowed information previously written in HTML format files to be served.


## Second generation of the WWW: dynamic content

In the next generation, content was generated partially or totally dynamically:
 * On the server with technologies such as servlets, JSP, PHP, etc.
 * On the client with technologies such as Applets, JavaScript, etc.

There is an interesting additional step: what if behind a dynamic file there is not a file but a program to which parameters are sent and which responds with the result?
The POST method was used to send useful information for dynamic generation: the “parameters” to work with.

Therefore, we have the potential to:
* When writing:
  ```
  GET /calculator,add,2,3 HTTP/1.0
  accept: */*
  ```
* Have the following output instead of HTML text:
  ```html
  HTTP/1.1 200 OK
  Date: Fri, 28 Apr 2023 11:18:58 GMT
  Server: Apache
  Location: https:///
  Content-Length: 1
  Connection: close
  Content-Type: text/html; charset=iso-8859-1

  5
  ```

Initially, this "dynamism" was used to allow people to enter search terms and have a results page associated with those terms built dynamically.

The second generation was initially characterized by the following:
  * There were different technologies that were not always compatible.
  * It was very oriented towards interaction with people.
  * For security reasons, firewalls usually only allowed traffic to port 80.


## Why use a Web Service

However, if this dynamic generation mechanism is used to communicate between applications, we have the basis for building a possible Web service.
In the third generation:
 * The dynamic content generation mechanism began to be used for b2b (*business to business*).
 * Different standards appeared that allow us to build web services in which incompatibilities are avoided as much as possible.
 * A web service began to be defined based on dynamic content generation, facilitating the deployment of a new generation of distributed applications.

A **web service** is a set of protocols and standards used to exchange data between applications on computer networks such as the Internet:
   * **Interoperability** is achieved through the adoption of **open standards**.
     * The OASIS and W3C organizations are the committees responsible for the architecture and regulation of web services.


## Service styles: SOAP vs. REST

There are two main styles:
  * **SOAP** (*Simple Object Access Protocol*) web services
  * **REST** (*RESTful Architecture Style*)

The idea of Web Services first appeared and was standardized with the use of **SOAP**, which offers many functionalities for *business to business*.
Over time, another complementary need arose: a simplified version that would allow for the definition of a simple Web API.
In his doctoral thesis in 2000, Roy Fielding introduced the REST proposal, which provides a relatively high level of flexibility, scalability, and efficiency for developers.
REST APIs have become popular, being the common method for connecting components and applications in a microservices architecture.

<html>
<table>
<tr><th></th><th>SOAP</th><th>REST</th></tr>
<tr>
<td>Orientation</td>
<td>Message-oriented</td>
<td>Resource-oriented</td>
</tr>
<tr>
<td>A URL represents...</td>
<td>Access to an API</td>
<td>A resource</td>
</tr>
<tr>
<td>HTTP methods</td>
<td>Usually only POST is required (and in some simple cases GET can be used)</td>
<td>POST and/or PUT/GET/UPDATE/DELETE correspond to CRUD (Create/Read/Update/Delete)</td>
</tr>
<tr>
<td>Data representation</td>
<td>XML</td>
<td>JSON (but also HTML, plain text, XML)</td>
</tr>
<tr>
<td>Advantages</td>
<td>
<li> Security</li>
<li> Flexibility</li>
</td>
<td>
<li> Requires less bandwidth</li>
<li> Stateless technology</li>
<li> Caching</li>
<li> Scalability</li>
</td>
</tr>
<tr>
<td>Registration</td>
<td><img src="/materials/topic-ws/web-services/web-services-drawio_soap.svg"></td>
<td><img src="/materials/topic-ws/web-services/web-services-drawio_rest.svg"></td>
</tr>
</table>
</html>

The [six design principles (or architectural constraints)](https://www.ibm.com/topics/rest-apis) of REST are:
* Uniform interface: All API requests for the same resource should look the same, regardless of where the request comes from.
* Client-server decoupling: Client and server applications should be completely independent of each other, interacting only with the API.
* Stateless: REST APIs are stateless, which means that each request must include all the information necessary to process it.
* Cacheability: As much as possible, resources should be cacheable on the client or server.
* Layered architecture: Calls and responses can pass through different layers (do not assume that the client and server connect directly to each other).
* Code on demand (optional): REST APIs typically send static resources, but in some cases, responses may also contain executable code (such as Java applets). In these cases, the code should only be executed on demand.


### Example of communication with SOAP and REST

For the following function:
  ```c
  Float price ;
  Price = GetPrice(table);
  ```

The following table shows an example of what the requests and responses might look like:

<html>
<table>
<tr><th></th><th>SOAP</th><th>REST</th></tr>

<tr>
<td>
Request
</td>
<td>
<pre>
POST StockQuote HTTP/1.1
...
&lt;SOAP-ENV:Envelope
  xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
  
SOAP-ENV:encodingStyle =“http://schemas.xmlsoap.org/soap/
&lt;SOAP-ENV:Body&gt;
  &lt;m:GetPrice
     xmlns:m="http: //example.com/stockquote.xsd"&gt;
     &lt;item&gt;table&lt;/item&gt;
  &lt;/m:GetPrice&gt;
&lt;/SOAP-ENV:Body&gt;
&lt;/SOAP-ENV:Envelope&gt;
</pre>
</td>
<td>
<pre>
POST /price HTTP/1.1
Content-Type: application/json
...
{"item":"table"}
</pre>
</td>
</tr>

<tr>
<td>
Response
</td>
<td>
<pre>
HTTP/1.1 200 OK
...
&lt;SOAP-ENV:Envelope
  xmlns:SOAP-ENV="http://schemas.xmlsoap.org/soap/envelope/"
  SOAP-ENV:encodingStyle ="http://schemas.xmlsoap.org/soap/
&lt;SOAP-ENV:Body&gt;
   &lt;m:GetPriceResponse
     xmlns:m="http://example.com/stockquote.xsd"&gt;
     &lt;Price&gt;123.5&lt;/Price&gt;
   &lt;/m:GetPriceResponse&gt;
&lt;/SOAP-ENV:Body&gt;
&lt;/SOAP-ENV:Envelope&gt;
</pre>
</td>
<td>
<pre>
HTTP/1.0 201 CREATED
...
123.5
</pre>
</td>
</tr>

<tr>
<td>
Main technologies (protocols, etc.)
</td>
<td>
<ul>
<li> HTTP: transport used</li>
<li> XML: describes the information, messages</li>
<li> SOAP: packages the information and transmits it between the client and the service provider</li>
<li> WSDL: service description</li>
<li> UDDI: list of available services </li>
<ul>
</td>
<td>
<ul>
<li> HTTP: transport used </li>
<li> JSON: describes information as a data structure in JavaScript</li>
<ul>
</td>
</tr>

<tr>
<td>
Relationship between technologies
</td>
<td>
<img src="/materials/topic-ws/web-services/web-services-drawio_10_soap.svg" alt="Typical protocol stack in SOAP-based web services">
</td>
<td>
<img src="/materials/topic-ws/web-services/web-services-drawio_10_rest.svg" alt="Common protocol stack in REST-based web services">
</td>
</tr>

</table>
</html>


##  Service examples using REST and SOAP

Below are three simple examples in Python:
 * REST web service (server and client)
 * SOAP web service (client only)
 * SOAP web service (client and server)

And then two examples in C:
  * Using a gSOAP/XML-based service (client only)
  * Creating a service based on gSOAP/XML (client and server)


## Simple example of a REST web service (server and client in Python)

The following example implements a web service server in Python that allows you to check the price of a table in a restaurant:

* The **`app.py`** script implements a web service in Python using the *Flask* and *request* packages:
  ```python
  from flask import Flask, request

  app = Flask(__name__)
  prices = { "table": "123.5", "reservation": "12.5" }

  @app.route('/price', methods=["POST"])
  def add_price():
       try:
          req  = request.get_json()
          item = req['item']
          return prices[item], 201
       except Exception as e:
          return {"error": str(e)}, 415

   app.run(debug=False, host="0.0.0.0", port="5000")
   ```

* To *start* the server, use the following:
  ```bash
  python3 app.py &
  ```
 
* To *stop* the server (when the service is no longer needed), use the following:
  ```bash
  kill $(pgrep -f app.py)
  ```


The following example implements a client for the above web service using Python:

* The **`clnt.py`** script uses *requests*:
  ```python
  import requests

  r = requests.post(url="http://127.0.0.1:5000/precio",
                     json={ "item": "table" },
                     headers={ 'Content-type': 'application/json' })
  print(r.text)
  ```

* To run the client, you can use:
  ```bash
  python3 clnt.py
  ```

* The output is:
  ```bash
  123.5
  ```

It is even possible to use the `curl` command as a client for the previous web service:
 
* To run a client, you can use:
   ```bash
   curl -i http://127.0.0.1:5000/precio \
        -X POST \
        -H 'Content-Type: application/json' \
        -d '{"item":"table"}'
   ```

 
* And the output would be:
  ```bash
  127.0.0.1 - - [11/Mar/2025 12:42:00] "POST /price HTTP/1.1" 201 -
  HTTP/1.0 201 CREATED
  Server: Werkzeug/2.3.2 Python/3.10.12
   
  Date: Tue, 11 Mar 2025 11:42:00 GMT
  Content-Type: text/html; charset=utf-8
  Content-Length: 5
  Connection: close

  123.5
  ```






<br>

### OpenAPI

Based on the initial example above, we can ask ourselves some questions, such as:
* How are we going to document this REST service?
* Could it be described in a simplified WSDL format to generate the documentation automatically?
* Could it be described in a simplified WSDL format to generate the code automatically?

The OpenAPI standard seeks to help with the above questions because:
* It allows you to describe a REST service:
   * The OpenAPI 3.1 standard documentation is available at: https:/ /spec.openapis.org/oas/v3.1.1
   * It includes examples such as:
     * Simple example: https://learn.openapis.org/examples/v3.0/link-example.html
     * Pet store: https://github.com/swagger-api/swagger-petstore/blob/master/src/main/resources/openapi.yaml
* Based on this standard, there are tools to help with the design, development, and testing of such REST services:
  * To design the API, you can use an online editor such as Swagger Editor: https://editor.swagger.io/
  * To generate code (client libraries, server stubs, documentation) in different languages, you can use Swagger Codegen: https://github.com/swagger-api/swagger-codegen


### Simple example of a REST web service with OpenAPI

The following example implements a small key-value storage service:

0. First, you need to check that the Python packages **fastapi, uvicorn, and pydantic** are installed.
   You can install them using:
   ```bash
   pip3 install fastapi uvicorn[standard] pydantic
   ```
   Or using:
   ```bash
   sudo python3 -m pip install fastapi uvicorn[standard] pydantic
   ```
   The installation usually takes some time, so you will need to wait.
   You can use ```pip3 install --user fastapi uvicorn pydantic``` to install only for the current user.
1. The next step would be to create the file **``ws-openapi.py``** with the server for that web service:
   ```python
   from fastapi  import FastAPI
   from pydantic import BaseModel
   import uvicorn

   # Item associated with each key
   class Item(BaseModel):
       name: str
       weight: float

   # {10, {name:'', weight:0.0}, ...
   list_kv = {}

   # fastAPI
   app = FastAPI()

   @app.put("/items/{key}")
   async def add_item(key: int, item: Item):
       list_kv[key] = item
       return { 'key': key, "item": item}

   @app.get("/items/{key}")
   async def select_item(key: int):
       try:
          item = list_kv[key]
       except:
          item = { '', 0.0 }
       return { 'key': key, "item": item }

   # Main
   if __name__ == "__main__":
      uvicorn.run(app, host="127.0.0.1", port=8000)
   ```
   The process of creating the above file has three main steps:
   <html>
   <table>
   <tr>
   <td>(1) Initial simple key-value API</td>
   <td>(2) Add pydantic</td>
   <td>(3) Add fastAPI and uvicorn</td>
   <tr>
   <tr>
   <td>
    
   ```python


   
   # Item associated with each key
   class Item:
      name: str
      weight: float

   # {10, {name:‘’, weight:0.0}, ...
   list_kv = {}

   # API
   
   def add_item(k: int, v: Item):
       list_kv[k] = v
       return { "key": k,
                "item": v }

   def select_item(key: int):
       try:
          item = list_kv[key]
       except:
          item = { '', 0.0 }
       return { "key": key,
                "item": item }
   
   # Main
   if __name__ == "__main__":
       print(add_item(10, {'name', 0.0}))
       print(select_item(10))
   ```

   </td>
   <td>
    
    ```python
    from pydantic import BaseModel
    
    # Item associated with each key
    class Item(BaseModel):
       name: str
       weight: float

   # {10, {name:'', weight:0.0}, ...
   list_kv = {}
   
   # API

   def add_item(k: int, v: Item):
       list_kv[k] = v
       return { "key": k,
                "item": v }



   def select_item(key: int):
       try:
          item = list_kv[key]
       except:
          item = { '', 0.0 }
       return { "key": key,
                'item': item }

   # Main
   if __name__ == "__main__":
       print(add_item(10, {'name', 0.0}))
       print(select_item(10))

   ```

   </td>
   <td>
         
   ```python
   from fastapi  import FastAPI
   from pydantic import BaseModel
   import uvicorn

   # Item associated with each key
   class Item(BaseModel):
       name: str
       weight: float

   # {10, {name:'', weight:0.0}, ...
   list_kv = {}
   
   # fastAPI
   app = FastAPI()

   @app.put("/items/{key}")
   async def add_item(k: int, v: Item):
       list_kv[k] = v
       return { 'key': k,
                "item": v }
   
   @app.get("/items/{key}")
   async def select_item(key: int):
       try:
          item = list_kv[key]
       except:
          item = { '', 0.0 }
       return { "key": key,
                'item': item }

   # Main
   if __name__ == "__main__":
      h="127.0.0.1"
      p=8000
      uvicorn.run(app, host=h, port=p)
   ```

   </td>
   </tr>
   </table>
   </html>
2. To run the server:
   ```bash
   python3 ws-openapi.py &
   ```
   And the output will look like:
   ```bash
   INFO:     Started server process [67103]
   INFO:     Waiting for application startup.
   INFO:     Application startup complete.
   INFO:     Uvicorn running on http://127.0.0.1:8000 (Press CTRL+C to quit)
   ```
3. To view the *swagger* interface with documentation and a basic interactive client, use a web browser:
   ```bash
   firefox http://127.0.0.1:8000/docs
   ```
    
This interface allows you to view the automatically generated documentation and interact with the service:
   ![Swagger interface for ws-openapi](/materials/topic-ws/ssdd_web-services/ws-openapi-ui.png)

   You can see that there is a link to http://127.0. 0.1:8000/openapi.json
   This web page allows you to access the OpenAPI description of the implemented service.

With the OpenAPI-based description of the REST service, there are tools that allow you to generate client (and even server) code in different languages.
As an example of the steps to have a simple Python client for the above key-value store service in a non-automatic way:

1. The first step is to create the client file **``ws-openapi-clnt.py``** for that web service:
   ```python
   import requests

   def add_item(key, value):
       http_url = "http://127.0.0.1:8000/items/" + str(key)
       http_headers = { 'Content-type': 'application/json', 'accept': 'application/json' }
       r = requests.put(url=http_url, json=value, headers=http_headers)
       return r.text

   key   = 10
   value = { "name": 'name', "weight": 0.0 }
   r = add_item(key,value)
   print(r)
   ```
2. To run this client, use:
   ```bash
   python3 ws-openapi-clnt.py
   ```
3. And the output could be similar to:
   ```bash
   {"key":10,"item":{"name":'name',"weight":0.0}}
   ```
You can see that the ``add_item`` function in the client encapsulates the call to the REST service.

<br>

More examples of use can be found in the following interesting articles:
* https://data-ai.theodo.com/en/technical-blog/fastapi-pydantic-powerful-duo
* https://medium.com/codenx/fastapi-pydantic-d809e046007f
* https://www.geeksforgeeks.org/fastapi-pydantic/
* https://realpython.com/fastapi-python-web-apis/

I would like to take this opportunity to thank Pablo Velo Moreno for introducing me to the world of fastAPI and pydantic.



## Simple example of a SOAP web service (Python client)

When creating a web service in Python with SOAP, there are multiple environments:
*https://wiki.python.org/moin/WebServices

The best-known examples are:
* **Zeep**: for creating **clients**
   * SOAP-based model for Python.
   * Uses Python dictionaries for XML handling.
* **Spyne** : for creating **services**
   * SOAP-based model for Python for service creation.
   * Server deployment.
   * WDSL generator.

The following example implements a greeting client (``Hello + <name>``) based on Zeep:

0. First, you need to check that the **zeep** Python package is installed.
   It can be installed using:
   ```bash
   pip3 install zeep
   ```
   The installation usually takes some time, so you will need to wait.
   You can use ```pip3 install zeep --user``` to install only for the current user.

1. You need to know the information about the web service using ``python -mzeep <URL>``, where URL is the one associated with the WSDL:
   ```bash
   python3 -mzeep https://apps.learnwebservices.com/services/hello?WSDL
   ```

2. The next usual step is to create the client file for that web service (ws-echo.py in our example):
   ```python
   import zeep

   wsdl = 'https://apps.learnwebservices.com/services/hello?WSDL'
   client = zeep.Client (wsdl=wsdl)
   print(client.service.SayHello('Web Services'))
   ```

3. To execute:
   ```bash
   $ python3 ./ws-echo.py
   Hello Web Services!
   ```

You can try other WSDL services available for free (at least at the time of writing this tutorial) from the following list:
* https://ngrashia.medium.com/compiled-list-of-free-and-working-wsdl-urls-a7a54be6b91
<br>


## Simple example of a SOAP web service (client and server in Python)

When creating a web service in Python with SOAP, there are multiple environments:
*https://wiki.python.org/moin/WebServices

The best-known examples are:
* **Zeep**: for creating **clients**
   * SOAP-based model for Python.
   * Uses Python dictionaries for XML handling.
* **Spyne**: for creating **services**
   * SOAP-based model for Python for service creation.
   * Server deployment.
   * WDSL generator.

The following example implements an echo client (repeat what is sent) based on Spyne:

0. First, you need to check that the Python packages **spyne** and **zeep** are installed.
   They can be installed using:
   ```bash
   pip3 install spyne zeep
   ```
   The installation usually takes some time, so you will need to wait.
   You can use ```pip3 install spyne zeep --user``` to install only for the current user.

1. The usual first step is to create the server file for that web service (ws-calc-server.py in our example):
   ```python
   import time, logging
   from wsgiref.simple_server import make_server
   
   from spyne import Application, ServiceBase, Integer, Unicode, rpc
   from spyne.protocol.soap import Soap11
   from spyne.server.wsgi import WsgiApplication

   class Calculator(ServiceBase):
       @rpc(Integer, Integer, _returns=Integer)
       def add(ctx, a, b):
           return a+b
       @rpc(Integer, Integer, _returns=Integer)
       def sub(ctx, a, b):
           return a-b

   application = Application(services=[Calculator],
                             tns=‘http://tests.python-zeep.org/’,
                             in_protocol=Soap11(validator=‘lxml’),
                             out_protocol=Soap11())
   application = WsgiApplication(application)

   if __name__ == ' __main__':
       logging.basicConfig(level=logging.DEBUG)
       logging.getLogger(‘spyne.protocol.xml’).setLevel(logging.DEBUG)
       logging.info("listening to http:// 127.0.0.1:8000; wsdl is at: http://localhost:8000/?wsdl ")
       server = make_server(‘127.0.0.1’, 8000, application)
       server.serve_forever ()
   ```

2. The second step is to run the server:
   ```bash
   python3 ./ws-calc-server.py
   ```

3. The next step is to find out the Web service information using ``python -mzeep <URL>``:
   ```bash
   $ python3 -mzeep http://localhost:8000/?wsdl
   ...
   Service: Calculator
     Port: Application (Soap11Binding: {http://tests.python-zeep.org/}Application)
         Operations:
            add(a: xsd:integer, b: xsd:integer) -> addResult: xsd:integer
            sub(a: xsd:integer, b: xsd:integer) -> subResult: xsd:integer
   ```

5. The next usual step is to create the client file for that web service (ws-calc-client.py in our example):
   ```python
   import zeep

   wsdl = “http://localhost:8000/?wsdl”
   client = zeep.Client(wsdl=wsdl)

   r = client.service.add(5, 2)
   print(‘ 5+2 = ’ + str(r))

   r = client.service.sub(5, 3)
   print (‘ 5-3 = ’ + str(r))
   ```

6. To run the service client:
   ```bash
   $ python3 ./ws-calc-client.py
   5+2 = 7
   5-3 = 2
   ```


## Using a distributed service based on gSOAP/XML (client only, in C)

We will use an example based on the calculator service available at: http://www.genivia.com/calc.wsdl

The main steps to follow are as follows:

0. First, you need to check that the **gsoap** and **libgsoap-dev** packages are installed.
   They can be installed using:
   ```bash
   sudo apt install gsoap gsoap-doc libgsoap-dev
   ```
1. First, you need to obtain the calc.h file with the service interface from the service description given in WSDL:
   ```bash
   wsdl2h -c -o calc.h http://www.dneonline.com/calculator.asmx?WSDL
   ```
2. Next, generate the stubs from the calc.h interface:
   ```bash
   soapcpp2 -CcL calc.h
   ```
3. Create the **`app -d.c`** that uses the service:
   ```c
   #include "CalculatorSoap.nsmap"
   #include "soapH.h"
   
   int main ( int argc, char *argv[] )
   {
      struct _tempuri__Add in ;
      struct _tempuri__AddResponse out ;
      struct soap *soap ;

      soap = soap_new();
      if (NULL == soap) { return -1; }
      
      in.intA = 1 ;
      in.intB = 1 ;
      int ret = soap_call___tempuri__Add(soap, NULL, NULL, &in, &out) ;
      if (SOAP_OK != ret) { soap_print_fault(soap, stderr); exit(-1); }

      printf("Sum = %d\ n", out.AddResult);

      soap_destroy(soap); soap_end(soap); soap_free(soap);
      return 0;
    }
   ```
4. Everything must be compiled, for example using:
   ```bash
   gcc -o app-d   app-d.c soapC.c soapClient.c -lgsoap
   ```
   If, for example, you have gsoap 2.8.127 installed with *brew*, you could use:
   ```bash
   gcc -o app-d \
        -I/opt/homebrew/Cellar/gsoap/2.8.127/include/ \
        -L/opt/homebrew/Cellar/gsoap/2.8.127/lib/ \
        app-d.c soapC.c soapClient.c -lgsoap
   ```
5. The example would be executed as follows:
   ```bash
   $ ./app-d
   Sum = 2
   ```


## Creating a distributed service based on gSOAP/XML (client and server, in C)

The following example is based on the calculator example available at: https://www.genivia.com/dev.html#client-c

In the process of creating a distributed service based on gSOAP/XML that allows addition and subtraction, <br>the steps to follow are:

1. Generate the header file **`calc.h`** with the service interface:
   ```c
    //gsoap ns service name: calc
    //gsoap ns schema namespace: urn:calc
    int ns__add(double a, double b, double *result);
    int ns__sub(double a, double b, double *result);
   ```
2. With *soapcpp2* (version 2.8.91 or compatible), generate the stubs from the calc.h interface:
   ```bash
   soapcpp2 -cL calc.h
   ```
   
Thanks to the soapcpp2 command, much of the work has been generated, as can be seen in the following figure: <br/>
   <img src="https://www.genivia.com/images/flowchart.png" style="max-height:512" /><br/><br/>
3. The part that implements the service must be created, for example in the file **`lib-server.c`**:
   ```c
   #include “soapH.h”
   #include “calc.nsmap”

   int main(int argc, char **argv)
   {
      struct soap soap;

      if (argc < 2) {
          printf(“Usage: %s <port>\n”, argv[0]) ;
          exit(-1);
      }
      soap_init(&soap);
      soap_bind(&soap, NULL, atoi(argv[1]), 100);
      while(1)
      {
        soap_accept(&soap);
        soap_serve(&soap);
        soap_end(&soap);
      }
      
      return 0;
   }

   int ns__add ( struct soap *soap, double a, double b, double *result )
   {
       *result = a + b;
       return SOAP_OK;
   }

   int ns__sub ( struct soap *soap, double a, double b, double *result )
   {
       *result = a - b;
       return SOAP_OK;
   }
   ```
4. Create the **`app-d.c`** application that uses the service:
   ```c
   #include "soapH.h"
   #include "calc.nsmap"

   int d_add ( double a, double b, double *r )
   {
       struct soap soap;
       
       // env SERVER_IP=localhost:12345 ./app-d
       char *host = getenv("SERVER_IP") ;
       if (NULL == host) {
           printf("ERROR: missing ‘export SERVER_IP=<ip>:<port>’\n") ;
           return -1 ;
       }
       
       soap_init(&soap);
       soap_call_ns__add(&soap, host, “”, a, b, r);
       if (soap.error) {
           soap_print_fault(&soap, stderr);
           return -1 ;
       }
       soap_done(&soap) ;

       return 0 ;
   }
   
   int main ( int argc, char **argv )
   {
       double result = 0 ;
       int ret ;

       ret = d_add(1.0, 2.0, &result) ;
       if (ret < 0)
            printf("result is not valid\n") ;
       else printf("result = %g\n", result);

       return 0;
   }
   ```
5. Compile everything, for example using:
   ```bash
   gcc -g -o app-d       app-d.c soapC.c soapClient.c -lgsoap
   gcc -g -o lib-server  lib-server. c soapC.c soapServer.c -lgsoap
   ```
   If, for example, you have gsoap 2.8.127 installed with *brew*, you could use:
   ```bash
    gcc -g -o app-d \
        -I/opt/homebrew/Cellar/gsoap/ 2.8.127/include/ -L/opt/homebrew/Cellar/gsoap/2.8.127/lib/ \
        app-d.c soapC.c soapClient.c -lgsoap
    gcc -g -o lib-server \
        -I/opt/homebrew/Cellar/gsoap/2.8.127/include/ -L/opt/homebrew/Cellar/gsoap/2.8.127/lib/ \
        lib-server.c soapC.c soapServer.c -lgsoap
   ```
6. It is possible to run the server (lib-server) on one side and the client (app-d) on the other as follows:
   ```bash
   $ ./lib-server 12345 &
   $ env SERVER_IP=localhost:12345 ./app-d
   result = 3
   ```


## Other technologies besides REST and SOAP

REST and SOAP have evolved, and new technologies have emerged.
The following image summarizes the evolution over time and compares the main technologies:

 ![Evolution of Web Technologies](/materials/topic-ws/web-services/web-services-api-timeline.svg) <br/ >

The following table summarizes the main aspects of some of the main technologies:

|                 | SOAP        | REST           | JSON-RPC   | gRPC | GraphQL | Thrift |
|-----------------|-------------|----------------|------------|------|---------|--------|
| Introduced      | late 1990s   | 2000           | mid-2000s | 2015 | 2015    | 2007   |
| Format         | XML         | JSON, XML, ... | JSON       | Protobuf, JSON, ... | JSON | JSON or binary |
| Advantages        | Well-known    | Flexible and simple exchange format | Simple to implement | Allows any type of function | Flexible data structuring | Adaptable to many use cases |
| Use cases    | Payments, " legacy," ...   | Public API |        | HPC microservices | Mobile API |  |
| Organization    | Packaged messages | Meet 6 restrictions |        | Procedure call | Typed system |  |

Two of these technologies are particularly noteworthy:
* **gRPC**, a project created by Google for internal use.
* **Apache Thrift**, which is a project originally created by Facebook also for internal use.
Both technologies allow you to create software in different languages and use efficient protocols.


### Driving force behind the evolution: services with greater needs

Today, the most widely used standard as the basis for web services is HTTP, which means that using only HTTP, everything has to be based on the * request/response* interaction model and a protocol based on *text* messages.
In some cases, this may not be the ideal model for building certain services, forcing those services to be adapted to the model and protocol in a way that complicates both the design and the efficiency of execution.

Some examples of interaction models other than *request/response* are:
* **Fire-and-forget**. This model is an optimization of the *request/response* model that is useful when a response is not necessary. For example, if a customer wants to use a non-critical event logging service . It saves both network resources (by not sending the response) and processing resources (by not needing to process the response). It is also commonly used in [messaging systems](https://dasunpubudumal.medium.com/distributed-systems-messaging-eee8e70c4a77).
* **Request/stream**. This model is a variant of *request/response* in which the response is not a single message but a collection of messages. For example, if a client requests the list of products in a catalog.
* **Event**. This model is a variant of the *request/response* model in which there is no request, but rather a sequence of  responses, which are response messages associated with asynchronous events. For example, if a customer wants to subscribe to comments made during a sports broadcast.


# # Examples of other technologies

The following are examples of other technologies:
* Python - SSE: Simple example of an SSE-based web service
* BASH - SSE: Simple example of an SSE-based web service
* Creating a distributed service based on Apache Thrift (client and server, in Python)
* Creating a distributed service based on gRPC (client and server, in Python)

## Simple example of an SSE-based web service (in Python)

The following is an example of [Server Side Events](https://developer.mozilla.org/es/docs/Web/API/Server-sent_events/Using_server-sent_events) based on Python. \
Events sent by the server (SSE) is a technology that allows notifications/messages/events to be sent from the server to clients via an HTTP connection (push technology).

This example consists of two files:
* The **`demo-server.py`** file: a server that generates a message with the current time every second
   ```python
   import time, datetime
   from flask import Flask, Response
   from flask_cors import CORS

   app = Flask(__name__)
   CORS(app)
   
   @app.route(“/”)
   def publish_data():
       def stream():
           while True:
               ts  = datetime.datetime.now()
               msg = f“data: ” + str(ts) + “\n\n”
               yield msg
               time.sleep(1)

       return Response(stream(), mimetype="text/event-stream")

   if __name__ == “__main__”:
        app.run(debug=True, port=5000)
   ```
* The **`demo-client.html`** web page: displays the information as it arrives in real time.
  ```html
  <!DOCTYPE html>
  <html>
    <head><meta charset="utf-8"/></head>
     <body>
     <div id="msg1"></div><br>
     <div id="msg2" style="overflow-y: scroll;height: 300px; width: 360px;"></div><br>
     <script>
       var s = new EventSource(‘http://localhost:5000’);
           s.onmessage = function(e) {
               document.getElementById(“msg1”).innerHTML = e.data ;
               var old_msg2 = document.getElementById(“msg2”).innerHTML ;
               document.getElementById(“msg2”).innerHTML = e.data + ‘<br>’ + old_msg2 ;
           };
     </script>
     </body>
   </html>
  ```


The steps for typical execution are:
* Run the server:
  ```bash
  python3 ./demo-server.py
  ```
* Run the client:
  ```bash
  firefox demo-client.html
  ```


On the `demo-client.html` web page, an event arrives every second from `demo-server.py`.
Instead of displaying the time every second on the web page, other types of information (e.g., sensor values) can be sent, with different types of periodicity and different types of display or processing.


## Simple example of an SSE-based web service (in bash)

The following is an example of [Server Side Events](https://developer. mozilla.org/es/docs/Web/API/Server-sent_events/Using_server-sent_events) or server-sent events. \
Server-sent events (SSE) is a technology that allows notifications/messages/events to be sent from the server to clients via an HTTP connection (push technology).

This example is available at [ws-rest-sse-bash](/ws-rest-sse-bash/README.md) and consists of three files:
* The **`demo-server.sh`** script: sends the output of the **`demo.sh`** script to the **nc** (*net cat*) command listening on port 8080.
   
```bash
   #!/bin/bash
   set -x
   ./demo.sh | nc -l -p 8080
   ```
* The **`demo.sh`** script: sends the response headers from a web server, and then sends the time stamp in a JSON every second.
```bash
   #!/bin/bash
   
# (1) headers
   echo “HTTP/1.1 200 OK”
   echo “Access-Control-Allow-Origin: *”
   echo “Content-Type: text/event-stream”
   echo “Cache-Control: no-cache”
   echo ‘’
   # (2) every second it sends “data: {‘timestamp’: <instant>}”
   
while [ 1 ]; do
     T=$(date +%H:%M:%S)
     echo “data: {‘timestamp’: $T}”
     echo ‘’
     echo “”
     sleep 1
   done
   ```
* The **`demo-client.html`** web page: displays the information as it arrives in real time.
```html
   
<!DOCTYPE html>
   <html>
     <head><meta charset="utf-8" /></head>
     <body>
     <div id="msg1"></div><br>
     <div id="msg2" style="overflow-y: scroll;height: 300px; width: 360px;"></div><br>
     
<script>
       var s = new EventSource(‘http://localhost:8080’);
           s.onmessage = function(e) {
               document.getElementById(“msg1”).innerHTML = e.data ;
               var old_msg2 = document.getElementById(“msg2”).innerHTML ;
               
document.getElementById(“msg2”).innerHTML = e.data + ‘<br>’ + old_msg2 ;
           };
     </script>
     </body>
   
</html>
   ```


The steps for typical execution are:
* Run the server:
  ```
  chmod a+x *.sh
  ./demo-server.sh &
  ```
* Run the client:
  ```
  firefox demo-client.html
  ```


On the `demo-client.html` web page, every second an event arrives from the `demo.sh` script via the `nc` command, which simulates a web server on port 8080.
Instead of displaying the time every second on the web page, other types of information (e.g., sensor values) can be sent, with different types of periodicity and different types of display or processing.



## Creating a distributed service based on Apache Thrift (client and server, in Python)

The following example is based on the example available at: https://github.com/apache/thrift/tree/master/tutorial/py

In the process of creating a distributed service based on Apache Thrift that allows addition and subtraction, the steps to follow are:

1. Generate the **`calc.thrift`** file with the service interface:
   ```c
   service calc
   {
      i32 add(1:i32 a, 2:i32 b) ;
      i32 sub(1:i32 a, 2:i32 b) ;
   }
   ```
2. With *thrift* (version 0.13.0 or compatible), generate the stubs from the calc.thrift interface:
   ```bash
   thrift -r --gen py calc.thrift
   cd gen-py
   ```
3. You must create the part that implements the service, for example in the **`server.py`** file:
   ```python
   from calc import calc
   from calc.ttypes import *
   
   from thrift.transport import TSocket
   from thrift.transport import TTransport
   from thrift.protocol  import TBinaryProtocol
   from thrift.server    import TServer

   class calcHandler:
       def __init__(self):
           self. log = {}
       def add(self, n1, n2):
           print('add(%d,%d)' % (n1, n2))
           return n1 + n2
       def sub(self, n1, n2):
           print('sub(%d,%d)' % (n1, n2))
           return n1 - n2
   
   if __name__ == '__main__':
       handler   = calcHandler()
       processor = calc.Processor(handler)
       transport = TSocket.TServerSocket(host='127.0.0.1', port=9090)
       tfactory  = TTransport.TBufferedTransportFactory()
       pfactory  = TBinaryProtocol.TBinaryProtocolFactory()

       server = TServer.TSimpleServer(processor, transport, tfactory, pfactory)
       # You could do one of these for a multithreaded server
       # server = TServer.TThreadedServer  (processor, transport, tfactory, pfactory)
       # server = TServer.TThreadPoolServer(processor, transport, tfactory, pfactory)

       print('Starting the server on localhost:9090. ..')
       server.serve()
       print('done.')
   ```
4. Create the client part that uses the service, for example in the file **`client.py`**:
   ```python
   from calc import calc
   from calc.ttypes import *
   from thrift import Thrift
   from thrift.transport import TSocket
   from thrift.transport import TTransport
   from thrift.protocol import TBinaryProtocol

   def main():
       transport = TSocket.TSocket('localhost', 9090)
       transport = TTransport.TBufferedTransport(transport)
       protocol  = TBinaryProtocol.TBinaryProtocol (transport)

       transport.open()
       client = calc.Client(protocol)
       res_ = client.add(1, 1)
       print('1+1=%d' % res_)
       res_ = client.sub(1, 1)
       print('1-1=%d' % res_)
       transport.close()

   
   if __name__ == '__main__':
       try:
           main()
       except Thrift.TException as tx:
           print('%s' % tx.message)
   ```
5. You can run the server and client as follows:
   ```bash
   $ python3 ./server.py &
   $ python3 ./client.py
   add(1,1)
   1+1=2
   sub(1,1)
   1-1=0
   ```


## Creating a distributed service based on gRPC (client and server, in Python)

The following example is based on the example available at: https://medium.com/@coderviewer/simple-usage-of-grpc-with-python-f714d9f69daa

When creating a distributed service based on gRPC that allows addition and subtraction, <br>the steps to follow are:

1. Generate the **`calc.proto`** file with the service interface:
   ```c
   syntax = "proto3";
   package calculator;
   
   service Calculator {
     rpc Add (AddRequest) returns (AddResponse);
   }

   message AddRequest {
     int32 num1 = 1;
     int32 num2 = 2;
   }
   
   message AddResponse {
     int32 result = 1;
   }
   ```
2. With *grpc_tools.protoc* (``pip3 install grpcio grpcio-tools``), you must generate the stubs from the calc.proto interface:
   ```bash
   python3 -m grpc_tools.protoc -I. --python_out=. --grpc_python_out=. calc.proto
   ```
3. Create the part that implements the service, for example in the **`server.py`** file:
   ```python
   import grpc
   
   from concurrent import futures
   import calc_pb2
   import calc_pb2_grpc

   class CalculatorServicer(calc_pb2_grpc.CalculatorServicer):
        def Add(self, request, context):
            result = request.num1 + request.num2
            return calc_pb2.AddResponse(result=result)
   
   server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
   calc_pb2_grpc.add_CalculatorServicer_to_server(CalculatorServicer(), server)
   server.add_insecure_port(‘[::]:50051’)
   server.start ()
   server.wait_for_termination()
   ```
4. You need to create the client part that uses the service, for example in the **`client.py`** file:
   ```python
   import grpc
   import calc_pb2
   import calc_pb2_grpc

   with grpc.insecure_channel(‘localhost:50051’) as channel:
       stub = calc_pb2_grpc.CalculatorStub(channel)
       response = stub.Add(calc_pb2.AddRequest(num1=1, num2=2))
   print(f“Result: {response.result}”)
   ```
5. You can run the server and client as follows:
   ```bash
   $ python3 ./server.py &
   $ python3 ./client.py
   
   Result: 3
   ```


## Example of JSON-RPC over HTTP: simple calculator MCP server (client and server, in Python)

* A **MCP-based service mainly offers four types of functions**:
* *Tools*: utilities are functions that the client can invoke to perform actions or access external systems.
  * *Resources*: resources expose data sources that the client can read.
  * *Resource templates*: resource templates are parameterized resources that allow the client to request specific data.
* *Prompts*: prompts are reusable message templates to guide an LLM.

* To work with the MCP protocol, uvicorn and the ```FastMCP``` and ```FastAPI``` libraries will be used:
  ```bash
  sudo apt install uvicorn -y
  pip3 install fastmcp fastapi --break-system-packages
  ```

* An example consisting of two files is available at [ws-jsonrpc-mcp](/materials/lab-mcp-jsonrpc/README.md):
* The **```mcp_server_calc.py```** server in Python for a simple calculator:
  ```python
    from fastapi import FastAPI
    from fastmcp import FastMCP
    import uvicorn
      
    # A. Initialize FastMCP
    mcp = FastMCP(“calculator”)
    ## A.1. Utilities (*tools*)
    @mcp.tool()
    def add(a: int, b: int) -> int:
    ‘’“Add two numbers and return the result.”“”
    return a + b
      
    @mcp.tool()
    def sub(a: int, b: int) -> int:
        “”“Subtract two numbers and return the result.”“”
        return a - b
    @mcp.tool()
    def mul(a: int, b: int) -> int:
        “”“Multiply two numbers and return the result.”‘’
        return a * b
    @mcp.tool()
    def div(a: int, b: int) -> int:
        “”“Divide two numbers and return the result.”“”
        return a / b
  
    # A.2. Inputs (*prompts*)
    @mcp.prompt()
    def calculator_prompt(a: float, b: float, operation: str) -> str:
        if operation == “add”:
            return f“The result of adding {a} and {b} is {add(a, b)}”
        elif operation == “subtract”:
            return f“The result of subtracting {b} from {a} is {sub(a, b)}”
        elif operation == ‘multiply’:
            return f"The result of multiplying {a} and {b} is {mul(a, b)}“
        elif operation == ‘divide’:
            try:
                return f”The result of dividing {a} by {b} is {div(a, b)}"
            except ValueError as e:
                return str(e)
        else:
            return “Invalid operation. Please choose add, subtract, multiply, or divide.”
  
    # B. Initialize FastAPI
    mcp_app = mcp.http_app(path=“/”)
    api = FastAPI(lifespan=mcp_app.lifespan)
      
    ## B.1. Get status
    @api.get(“/api/status”)
    def status():
        return {“status”: ‘ok’}
    # C. Mount MCP on FastAPI at “/mcp”  and run on (IP=ANY, port=8000)
    api.mount(“/mcp”, mcp_app)
      
    if __name__ == “__main__”:
        uvicorn.run(api, host="0.0.0.0", port=8000)
    ```
* The Python client **```mcp_client_calc.py```** provides a simple calculator:
  ```python
    import asyncio
    
    from fastmcp import Client

    async def main():
    # Connect to the MCP server via HTTP
    client = Client(“http://127.0.0.1:8000/mcp/”)

    # Open session
    async with client:
    # List available tools
          
    tools = await client.list_tools()
    print(“Available tools:”)
    for tool in tools:
        print(“-”, tool.name)
        print(“\n”)
          
    # Call add(1,2)
    result = await client.call_tool(“add”, {“a”: 1, ‘b’: 2})
    print(“Result of add(1,2):”)
    print (result)

    if __name__ == “__main__”:
       asyncio.run(main())
  ```

### Example of execution:

<html>
<table>
<tr><th>Step</th><th>Client</th><th>Server</th></tr>
<tr>
<td>1</td>
<td></td>
<td>

```bash
$ python3 ./mcp_server_calc.py &

...
INFO: Started server process [172886]
INFO: Waiting for application startup.
INFO: Application startup complete.
INFO: Uvicorn running on http://0.0.0.0:8000
      (Press CTRL+C to quit)
```

</td>
</tr>

<tr>
<td>2</td>
<td>

```bash
$ python3 ./mcp_client_calc.py

...
Available tools:
  - add
  - sub
  - mul
  - div

Result of add(1,2):
CallToolResult(content=[TextContent(type=‘text’,
                                   text=‘3’,
                                   annotations=None,
                                   meta=None)],
                 
structured_content={‘result’: 3},
                 meta=None, data=3,
                 is_error=False)
...
```

</td>
<td>

```bash
...
INFO: 127.0.0.1:48316 - “POST /mcp/ HTTP/1.1” 200 OK
INFO: 127.0.0.1:48320 - “POST /mcp/ HTTP/1.1” 202 Accepted
INFO: 127.0.0.1:48336 - “GET /mcp/ HTTP/1.1” 200 OK
INFO: 127.0.0.1:48342 - “POST /mcp/ HTTP/1.1” 200 OK
INFO: 127.0.0.1:48358 - “POST /mcp/ HTTP/1.1” 200 OK
INFO: 127.0.0.1:48364 - “DELETE /mcp/ HTTP/1.1” 200 OK
...
```

</td>
</tr>

<tr>
<td>3</td>
<td></td>
<td>

```bash
$ killall python3

INFO: Shutting down
INFO: Waiting for application shutdown.
INFO: Application shutdown complete.
INFO: Finished server process [181555]
```

</td>
</tr>
</table>
</html>



<br/>

## Additional information
* Slides:
    * [Topic 8: t8_web-services.pdf](https://github.com/acaldero/uc3m_ds/blob/main/materials/topic-ws/t8_web-services.pdf)
* SOAP, REST, gRPC, etc.:
    * [Know your API protocols: SOAP vs. REST vs. JSON-RPC vs. gRPC vs. GraphQL vs. Thrift](https://www.mertech.com/blog/know-your-api-protocols)
    * [A brief look at the evolution of interface protocols leading to modern APIs](https://www.soa4u.co.uk/2019/02/a-brief-look-at-evolution-of-interface.html)
    * [Soap vs REST vs GrapQL vs gRPC](https://www.altexsoft.com/blog/soap-vs-rest-vs-graphql-vs-rpc/)
    * [A brief look at the evolution of interface protocols leading to modern APIs](https://www.soa4u.co.uk/2019/02/a-brief-look-at-evolution-of-interface.html)
    * [RPC Frameworks: gRPC vs Thrift vs RPyC for python](https://www.hardikp.com/2018/07/28/services/)
* MCP:
    * [Example of an MCP server](https://gofastmcp.com/tutorials/create-mcp-server)
    * [How to Build an MCP Server in Python: A Complete Guide](https://scrapfly.io/blog/posts/how-to-build-an-mcp-server-in-python-a-complete-guide)

