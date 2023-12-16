Federated Secure Computing
==========================

(C) 2023 by the authors. Licensee MDPI, Basel, Switzerland. This article is an open access article distributed under the terms and conditions of the [Creative Commons Attribution (CC BY) license](https://creativecommons.org/licenses/by/4.0/). Please cite as:

    Ballhausen H, Hinske LC. Federated Secure Computing. Informatics. 2023; 10(4):83. https://doi.org/10.3390/informatics10040083

by Hendrik Ballhausen (1,*) and Ludwig Christian Hinske (2,3)

(1) Medical Faculty, Ludwig-Maximilians-Universität München, Geschwister-Scholl-Platz 1, 80539 Munich, Germany

(2) Institute for Digital Medicine, University Hospital Augsburg, Stenglinstrasse 2, 86156 Augsburg, Germany

(3) Department of Anaesthesiology, LMU University Hospital, LMU Munich, Marchioninistrasse 15, 81377 Munich, Germany

(*) Author to whom correspondence should be addressed. 

Informatics **2023**, 10(4), 83; https://doi.org/10.3390/informatics10040083

**Submission received: 24 August 2023 / Revised: 22 October 2023 / Accepted: 23 October 2023 / Published: 31 October 2023**


# Abstract

Privacy-preserving computation (PPC) enables encrypted computation of private data. While advantageous in theory, the complex technology has steep barriers to entry in practice. Here, we derive design goals and principles for a middleware that encapsulates the demanding cryptography server side and provides a simple-to-use interface to client-side application developers. The resulting architecture, “Federated Secure Computing”, offloads computing-intensive tasks to the server and separates concerns of cryptography and business logic. It provides microservices through an Open API 3.0 definition and hosts multiple protocols through self-discovered plugins. It requires only minimal DevSecOps capabilities and is straightforward and secure. Finally, it is small enough to work in the internet of things (IoT) and in propaedeutic settings on consumer hardware. We provide benchmarks for calculations with a secure multiparty computation (SMPC) protocol, both for vertically and horizontally partitioned data. Runtimes are in the range of seconds on both dedicated workstations and IoT devices such as Raspberry Pi or smartphones. A reference implementation is available as free and open source software under the MIT license.

**Keywords: privacy-preserving computing; cloud computing; federated computing; cryptography; secure multiparty computation; propaedeutic framework; Python; free and open source software**


# 1. Introduction

Data has been called the “new oil” that fuels the digital future economy. Society and science need data to make informed decisions, and such information becomes more reliable when it builds upon independent data sources. Enterprises and consumers begin to understand the value of their data assets, and again, such data become more valuable when they are pooled from inaccessible silos into large comprehensive data lakes.

On the other hand, there is a lot of friction in sharing data openly. Companies are afraid to reveal trade secrets to their competitors. Research and development agencies closely guard the results of their work. Consumers and citizens are concerned about privacy and are wary of their data being potentially used to their disadvantage. Data protection and data security are ubiquitous, and informational self-determination has practically become a basic human right.

As a consequence, while 84% of companies believe analytics will improve their competitive position somewhat or significantly [1], and 75% of companies would be willing to share their data [2], only 39% of European companies claim to share data with other companies [3]. Similarly, 92% of internet users are concerned about privacy [4].

In other words, traditionally, there is a tradeoff between the value of data sharing and the need for privacy and data protection. Public domain data and open data have the highest societal and public economy benefits but require participants to relinquish their rights to their data and do not offer much in terms of ex-post control after consent. The middle ground is data sharing and collaboration, subject to licenses and contracts. Due to the ephemeral nature of data, these are often difficult to control and enforce. Finally, data may be privately owned with restricted and limited access by other parties.

There have always been attempts to create frameworks for data sharing that would provide more benefits with fewer downsides. For example, a classic analog example of a “trusted third party” would be the business consultant who confidentially learns the trade secrets of a pool of companies and redistributes the information as sanitized benchmarks and best practices among them.

In the information age, “federated computing” aims to integrate data from heterogeneous sources. Results are computed by the involved parties in a distributed fashion. In particular, privacy-preserving computation (PPC) aims to avoid open data sharing and safeguard the privacy of data subjects. Some of the more prominent examples include the following.

Secure multiparty computation (SMPC) [5,6] locally encrypts input data as cryptographic shares and then performs joint computation on those shares in a peer-to-peer network to derive a result that then becomes known to all involved parties. SMPC is often seen as a gold standard, potentially providing mathematically proven security, even in anonymous, trustless settings with some malicious parties that may seek to deviate from the protocol to discover the other parties’ secret inputs.

Fully homomorphic encryption (FHE) [7,8] locally encrypts input data before sending them to some public cloud. The cloud then performs computation on encrypted data only. While simple and powerful, only very few algorithms actually work on encrypted data.

Differential Privacy (DP) [9,10] seeks to limit access to some databases by keeping track of some “privacy budget”. In particular, researchers are limited in what they may learn about individual data points in the database. This is useful for scientific computation, e.g., on medical databases that seek to derive some aggregate statistics.

For an overview of related technologies, see [11]. To some degree, these technologies all share a common problem. Their power to reconcile data sharing with privacy comes at the cost of increased computational overhead and operational complexity. For example, in the case of SMPC, most frameworks aim to provide a monolithic yet universal solution, replacing basic arithmetic operations with exponentially growing garbled or arithmetic circuits [12,13] or costly asymmetric encryption [14,15,16].

There are only a few turn-key solutions commercially available, e.g., Sharemind [17]. Available open source frameworks, on the other hand, are rarely industry-grade. They rely on complex dependencies and are built on exotic tech stacks that are cumbersome to develop, operate, and secure. Few are simple to use and explicitly designed to work in propaedeutic settings, e.g., EasySMPC [18].

Presently, PPC still has several barriers to entry. Some are on the business side (lacking financial incentives and unproven business models) or are purely historical (lack of visible role models and successful showcases). These are out of the scope. However, many barriers are related to technical pain points that render the development, deployment, and operation of PPC solutions cumbersome and difficult. In particular, potential users and researchers note the computational overhead (often prohibitive in the case of monolithic universal solutions), exotic tech stacks and dependencies, the lack of cryptography skills of their developers, and the lack of pretext and understanding by their data protection officers. These can and should be addressed by the architecture.

In this work, we present an architecture that alleviates several of the perceived “pain points” and provides a simple yet effective framework to employ different PPC technologies. The goal is to separate the complexity of cryptographic protocols from business logic concerns. The minimalistic solution should be useable by small teams without a background in cryptography. Multiple topologies between data owners, processing nodes, and data consumers should be feasible. Developers should not be forced to use any particular language or libraries. Data flow should be transparent and straightforward to secure. Finally, the solution should be lightweight enough to work in IoT settings and scale to potentially large networks.


# 2. Methods

## 2.1. Pain Points and Design Goals

The following pain points are recurring topics in the literature, have been encountered in interviews with prospective early adopters, or have been experienced firsthand by the authors. Each has led to the formulation of a specific design goal addressing those concerns (see Table 1).

#### Table 1. Pain points and derived design goals.

<table cellspacing="0" cellpadding="0" border="1">
 <thead>
  <tr>
   <td><strong>Pain Points</strong></td>
   <td><strong>Design Goals</strong></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td colspan="2"><strong>related to capabilities (C)</strong></td>
  </tr>
  <tr>
   <td>front-end and business logic developers rarely have any expert knowledge of PPC</td>
   <td>provide high-level computing routines that hide low-level cryptography protocols <strong>(C1)</strong></td>
  </tr>
  <tr>
   <td>PPC is inaccessible to marginal groups lacking computing and personal resources</td>
   <td>build a minimalistic solution that can be run by a single developer on a Raspberry Pi [19] <strong>(C2)</strong></td>
  </tr>
  <tr>
   <td>PPC is difficult to teach and experience in the limited time frame of a typical lesson</td>
   <td>provide a propaedeutic solution that works in a school or university teaching setting <strong>(C3)</strong></td>
  </tr>
  <tr>
   <td colspan="2"><strong>related to development (D)</strong></td>
  </tr>
  <tr>
   <td>PPC often appears as the core functionality, so far as to even require to be a main routine</td>
   <td>PPC should be a network-level concern, separated from high-level concerns <strong>(D1)</strong></td>
  </tr>
  <tr>
   <td>introducing PPC functionality to a business logic often requires a complete rework</td>
   <td>enable piecewise introduction of PPC into an existing legacy business logic codebase <strong>(D2)</strong></td>
  </tr>
  <tr>
   <td>PPC frameworks require a specific tech stack and introduce a lot of dependencies</td>
   <td>client side and the core of the server side should be free of any dependencies <strong>(D3)</strong></td>
  </tr>
  <tr>
   <td>coding for any particular PPC framework locks the developer to a specific language</td>
   <td>let client-side developers freely choose their language or keep the legacy one <strong>(D4)</strong></td>
  </tr>
  <tr>
   <td colspan="2"><strong>related to security concerns (S)</strong></td>
  </tr>
  <tr>
   <td>some PPC frameworks are developed by non-experts in cryptography and are unsafe</td>
   <td>do not reinvent the wheel; make use of existing and proven PPC frameworks <strong>(S1)</strong></td>
  </tr>
  <tr>
   <td>PPC involves sensitive data that should not be visible to the front end or the outside</td>
   <td>enable topologies with data flow confined to trusted machines on the backend <strong>(S2)</strong></td>
  </tr>
  <tr>
   <td>many PPC use cases involve a third-party researcher who must be able to run analyses</td>
   <td>enable topologies with control flow coming from an external researcher <strong>(S3)</strong></td>
  </tr>
  <tr>
   <td>every PPC calculation requires a complete re-evaluation by data security officers</td>
   <td>separate topology, protocol, and function so they can be assessed independently <strong>(S4)</strong></td>
  </tr>
  <tr>
   <td colspan="2"><strong>related to deployment and operation (O)</strong></td>
  </tr>
  <tr>
   <td>PPC often requires all parties to agree on the exact same tech stack and IT environment</td>
   <td>enable joint computation between parties using different hardware or software <strong>(O1)</strong></td>
  </tr>
  <tr>
   <td>without a lot of experience, it is often unclear which PPC framework is best suited for a task</td>
   <td>frameworks should be replaceable without the need to rewrite business logic <strong>(O2)</strong></td>
  </tr>
  <tr>
   <td>universal PPC solutions often have enormous overhead in terms of space and processing</td>
   <td>provide “small and fast” solutions that cater to the most often encountered tasks <strong>(O3)</strong></td>
  </tr>
 </tbody>
</table>

## 2.2. Resulting Architecture

At its highest level, the architecture is dictated by a twofold separation: Business logic needs to be disentangled from cryptography protocols (separation of concerns, dependency inversion). And data flows of different data owners need to be separated from one another until they hit the underlying cryptography layer (privacy).

The first point is conveniently solved by a client/server architecture. Providing server-side PPC protocols through microservices and a web API to the client-side business logic addresses several of the above design goals.

However, if clients are to be entirely free of any cryptography concerns, they cannot send cryptographic shares to the server but need to be able to send raw data in the clear. This is also true because the client should be agnostic as to which particular PPC protocol is run by the server, as different protocols require quite different generations of shares.

Favoring minimalistic lean clients and simplicity of the client side over other considerations, we thus make the uncommon decision that every data owner runs their own server. In PPC lingo, we have as many compute nodes as there are input/data nodes, and they might coincide.
The resulting design decisions are listed in Table 2.

#### Table 2. Design decisions and their rationale.

<table cellspacing="0" cellpadding="0" border="1">
 <thead>
  <tr>
   <td><strong>Design Decision</strong></td>
   <td><strong>Rationale and Addressed Design Goals</strong></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>client–server architecture</td>
   <td>
    <ul>
     <li>offload computationally expensive cryptography to the server <strong>(D1)</strong></li>
     <li>separate business logic concerns (client-side) from cryptography concerns (server side) <strong>(C1, D1, O2)</strong></td></li>
    </ul>
  </tr>
  <tr>
   <td>one dedicated server per client</td>
   <td>
    <ul>
     <li>allows clients to remain lightweight and generalist API wrappers without specific encryption logic <strong>(D3)</strong></li>
     <li>servers can be trusted by their own client/data owner <strong>(S2, S3)</strong></li>
     <li>for less attack surface, it is possible to keep data entirely server side with only control flow coming from the client <strong>(S2, S3)</strong></li>
     <li>easy to analyze and straightforward to secure <strong>(S4, S2)</strong></li>
     <li>easy to explain and understand in a propaedeutic setting <strong>(C3)</strong></li>
  </tr>
  <tr>
   <td>solution should be a middleware</td>
   <td>
    <ul>
     <li>encapsulate low-level cryptographic scripts and provide them to the client as high-level macros <strong>(C1, D1)</strong></li>
     <li>provide the same macros for different PPC backends <strong>(O2, S4)</strong></li>
     <li>hosting one or several established PPC frameworks <strong>(S1)</strong></li>
     </li>
    </ul>
   </td>
  </tr>
  <tr>
   <td>provide microservices</td>
   <td>
    <ul>
     <li>rebuild business logic piecewise in a privacy-friendly fashion <strong>(D2)</strong></li>
     <li>provide highly optimized microservices for specific business problems instead of slow universal monoliths <strong>(O3, C2)</strong></li>
  </tr>
  <tr>
   <td>provide RESTful API</td>
   <td>
    <ul>
     <li>easy to provide API wrapper in any programming language <strong>(D4)</strong></li>
     <li>client remains free of any PPC-specific dependencies <strong>(D3)</strong></li>
    </ul>
   </td>
  </tr>
  <tr>
   <td>represent server-side objects 1:1 client-side</td>
   <td>
    <ul>
     <li>make client-side code easy to read and write <strong>(D1, C1, C3)</strong></li>
     <li>allow the client to remain lightweight yet powerful <strong>(D3, C2, C3)</strong></li>
     <li>allow server-side-only data flow <strong>(S2, S3, S4)</strong></li>
    </ul>
   </td>
  </tr>
  <tr>
   <td>implement server middleware in Python (optional)</td>
   <td>
    <ul>
     <li>core middleware can be implemented in pure Python without any additional dependencies except for a webserver <strong>(D3)</strong></li>
     <li>Python is available on most any platform <strong>(O1, C2)</strong></li>
     <li>Python is popular in propaedeutic settings and data science <strong>(C3)</strong></li>
    </ul>
   </td>
  </tr>
 </tbody>
</table>


## 2.3. Overall Concept of Federated Secure Computing

Federated Secure Computing connects heterogeneous systems (federation) through privacy-preserving computing protocols (secure computing), Figure 1.

#### Figure 1. Federated Secure Computing. 
![](images/concept1.png)

It achieves this as a middleware between client-side business logic and server-side cryptography backends. It encapsulates the secure computing functionality through specific microservices and exposes them to the clients through an easy-to-use API, Figure 2.

#### Figure 2. Flow of control, separation of concerns between client and server side, and API/microservice architecture. 
![](images/concept2.png)


## Client-Server Topologies

While the decision to have exactly one server per data owner seems to be quite strong, there really is no restriction on the actual topology of the PPC protocols employed in the backend.

In PPC lingo, there are <strong>data nodes</strong> (providing input data), <strong>compute nodes</strong> (running the protocols), and <strong>researcher nodes</strong> (providing control flow and receiving results). Here, we have <strong>server nodes</strong> and <strong>client nodes</strong> which may or may not coincide with data, compute, and/or researcher nodes.

### Example 1: clients act as data and researcher nodes, servers act as compute nodes

This topology is suitable when there are several equal and simultaneously active researchers in a symmetric peer-to-peer network.

#### Figure 1
![](images/example1.png)

Clients send unencrypted input data and control flow to their respective server. Servers host the PPC protocol and thus act as compute nodes. They break the input data into cryptographic shares and inject it into the protocol. They execute the protocol on encrypted shares and send the result back to their clients.

The propaedeutic protocol **SIMON** (**SI**mple **M**ultiparty computati**ON**) uses this topology.

### Example 2: servers act as data and compute nodes, single client as researcher node

The difference to Example 1 is that data is hosted on the server, not on the client. This is a more likely case in institutions where data is not supposed to be seen even by their own clients and researchers.

#### Figure 2

![](images/example2.png)

In this case, there is no need for more than one researcher (of course, having one researcher per server is still perfectly an option). The single researcher may send control flow to all servers (rendering synchronization trivial) and receive only the result of the computation (but has no access to input data on the servers).

Using server-side object representation, it is possible to write wrappers for server-side data base handles and access them through the client. Care must be taken in this case to properly secure the API, in particular by object level authorization.

This topology is suitable if there is a privileged researcher and a number of independent contributors. For example, a university hospital researching the data of teaching hospitals; a government agency using data of regional bodies; a parent company analyzing subsidiaries; an industry association providing benchmarks to their member companies; etc. 

### Example 3: servers run middleware only, additional compute nodes in the backend

Some PPC protocols might require a certain compute cluster of their own. For example, some SMPC protocols use three independent compute nodes, irrespective of the number of data nodes.

#### Figure 3

![](images/example3.png)

In this case, the role of the Federated Secure Computing servers is “only” to act as a gateway hosting a translatory middleware.

They middleware receives input data and translates it into cryptographic shares according to the protocol of the compute cluster; the middleware receives control flow from the client and accordingly instructs the compute cluster.

This topology is useful if one wants to combine the client-side simplicity of Federated Secure Computing with a more mature and complete solution to run the actual computations on the backend.

For example, a [Carbyne Stack](https://carbynestack.io) compute cluster would be a useful backend.

### Example 4: DataSHIELD

[DataSHIELD](https://www.datashield.org) is a popular PPC solution in academic and data science settings. It features a central compute node that receives aggregated data from data nodes, aggregates it further, and forwards the summary statistics to the researcher.

If one wants to capsulate the DataSHIELD server behind a Federated Secure Computing middleware, the topology will look like this figure:

#### Figure 4

![](images/example4.png)

In a way, this is a combination of Example 2 (a single researcher) and Example 3 (pure middleware functionality of Federated Secure Computing).

It might be useful if one wants to use client-side languages other than R to develop scripts; or it might be convenient to include DataSHIELD for its statistical power to process e.g. metadata of data that is analyzed in full by other protocols such as secure multiparty computation.

## Client-Side Stack

### Representation of server-side objects

Our design goal is to render client-side business-logic development as simple as possible. We do not want any specific dependencies client-side, and we would like to go through the API as transparent as possible. Hence, we would like to be able to write client-side code like this:

#### Code Listing 1 – Example of how client-side code should interact with server-side objects

```python
import federatedsecure.client

# connect to the server, return API handle
api = federatedsecure.client.connect(“https://my.server”)

# find a microservice that matches some requirements
microservice = api.create(functionality=“can do some stuff”})

# connect to some specific server-side database
database = api.create(connector=“myconnector”, version=“1.2.3”)

# fetch input data
data = database.get_handle().query(row=2, column=5)

# do some server-side computation
result = microservice.compute(data)

# download and output the result
print(api.download(result))
```

There are two functions that translate between server-side and client-side:

`api.create` returns a handle to a top-level server-side object, typically a microservice. It is given some arguments describing the desired microservice. 

`api.download` serializes the server-side data belonging to some handle and returns it to the client.

This means that all the other variables in above pseudocode (`microservice`, `database`, `data`, and `result`) are simply handles to server-side objects. The client can access them through the API and trigger server-side behavior without any client-side dependencies. We achieve this by using a wrapper class called `Representation`.

As its name implies, it represents server-side objects. It stores a pointer to the API (so it can trigger requests to the API) and a unique identifier (UUID) of the server-side object. Access to member variables and functions can then be reflected on the API by passing the UUID.

In Python, this is particularly straightforward:

#### Code Listing 2 – class Representation (simplified, see actual code!)

```python
class Representation:

  def __init__(self, api, uuid):
    self.api = api
    self.uuid = uuid

    def __getattr__(self, member_name):
      return self.api.attribute(self.uuid, member_name)

    def __call__(self, *args, **kwargs):
        return self.api.call(self.uuid, *args, **kwargs)
```

For example, `database.get_handle().query(row=2, column=5)` in fact creates four (!) representations: 1) of the member function `get_handle`, 2) of the result of invoking that member function without arguments, 3) of that result’s member function `query`, and 4) of the result of `query` with some arguments.

Note that such nice syntactic sugar is not available in all programming languages. For example, in R the same code would read:

#### Code Listing 3 – In some languages, client code will be more verbose than in Python

```r
source (“.../federatedsecure/client.r”)

# connect to the server, return API handle
api <- Api(“https://my.server”)

# find a microservice that matches some requirements
microservice <- api$create(kwargs=list(
                  functionality=“can do some stuff”))

# connect to some specific server-side database
database <- api$create(kwargs=list(
              connector=“myconnector”, version=“1.2.3”))

# fetch input data
func_handle <- database$attribute(“get_handle”)
handle <- func_handle$call()
func_query <- handle$attribute(“query”)
data <- func_query$call(list(row=2, column=5))

# do some server-side computation
func_compute <- microservice$attribute(“compute”)
result <- func_compute$call(list(data=data))

# download and output the result
print(api$download(result))
```

### API Wrapper

By using `Representation` the entire API traffic can be routed through very few RESTful endpoints, see Table 3.

#### Table 3 – API endpoints

<table>
 <thead>
  <tr>
   <td>verb</td>
   <td>endpoint</td>
   <td>server-side effect and response</td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>GET</td>
   <td>/representations</td>
   <td>
    <ul>
     <li>list of top-level microservices
    </ul>
   </td>
  </tr>
  <tr>
   <td>POST</td>
   <td>/representations</td>
   <td>
    <ul>
     <li>finds matching top-level microservice
     <li>returns uuid representing the microservice
    </ul>
   </td>
 </tr>
  <tr>
   <td>PUT</td>
   <td>/representations</td>
   <td>
    <ul>
     <li>upload data and stores it server-side
     <li>returns uuid representing the data
    </ul>
   </td>
 </tr>
  <tr>
   <td>PATCH</td>
   <td>/representation/{uuid}</td>
   <td>
    <ul>
     <li>calls server-side function represented by uuid
     <li>stores the return value server-side
     <li>returns uuid representing the return value
    </ul>
   </td>
 </tr>
  <tr>
   <td>GET</td>
   <td>/representation/{uuid}/{attr}</td>
   <td>
    <ul>
     <li>gets attribute (e.g. child variable, member function) of object represented by uuid
     <li>stores the pointer server-side
     <li>returns uuid representing the attribute
    </ul>
   </td>
 </tr>
  <tr>
   <td>GET</td>
   <td>/representation/{uuid}</td>
   <td>
    <ul>
     <li>serializes the object represented by uuid
     <li>returns the serialized data
    </ul>
   </td>
 </tr>
  <tr>
   <td>DELETE</td>
   <td>/representation/{uuid}</td>
   <td>
    <ul>
     <li>deletes the object represented by uuid
    </ul>
   </td>
  </tr>
 </tbody>
</table>

In these terms, it is easy to implement `api.create`, `api.download`, `api.call`, and `api.download` as used in the main routine and in  Representation above:

#### Code Listing 4 – class Api (simplified, see actual code!)

```python
class Api:

    def __init__(self, url):
        self.http = HttpInterface(url)

    def list(self):
        return self.http.GET('representations')

    def create(self, *args, **kwargs):
        response = self.http.POST('representations',
                   body={'args': args, 'kwargs': kwargs})
        return Representation(self, response['uuid'])

    def upload(self, *args, **kwargs):
        response = self.http.PUT('representations',
                   body={'args': args, 'kwargs': kwargs})
        return Representation(self, response['uuid'])

    def call(self, uuid, *args, **kwargs):
        response = self.http.PATCH('representation', uuid,
                   body={'args': args, 'kwargs': kwargs})
        return Representation(self, response['uuid'])

    def attribute(self, uuid, attr):
        response = self.http.GET('representation', uuid, attr)
        return Representation(self, response['uuid'])

    def download(self, representation):
        response = self.http.GET('representation',
                   representation.uuid)
        return response['object']

    def release(self, uuid):
        self.http.DELETE('representation', uuid)
        return None
```

### Client design considerations

**Programming language** - At the time of writing, there are API wrappers in Python, R, and JavaScript. As the client is very thin and only contains the classes Api and Representation and a HTTP interface, it is easy to develop API wrappers in other languages. And in principle, curl would suffice.

**Thin client** – One should keep the client as thin as possible. We want client-side developers to be free in their choice of language, so any functionality that we introduce in one language’s API wrapper we would have to introduce in each. Also, we want the client to be small (read: kilobytes) so it can be used in IoT settings.

**Macros should be server-side** – The last point implies that any macro functionality should not be written client-side. For example, if you want to combine several steps like connecting to a database, getting a handle, reading data and storing it into a single line of client-code, then you should write a small server-side extension for that. This way, the functionality will be available in all clients, and this eliminated several potentially slow and payload-heavy API calls. 

**Securing the API** – This is mainly a server-side concern, but one would probably have to account for authentication and authorization on the client, too. Federated Secure Computing is designed for propaedeutic settings, but if it were used in production, this would have to be adapted to the organization’s specific security implementation.

**Full RPC framework** – Our implementation above is minimalistic and propaedeutic. It is a lean way to interact with server-side objects in a generic way. In a production setting, you might want to employ a more complete and stable framework for remote procedure calls.

## Server-Side Stack

### Registry, Discovery, and Bus

The core middleware consists at the minimum of a registry of server-side objects, a bus to access them, and a discovery mechanism to register top-level objects.

The **registry** holds pairs of top-level microservices and their description. At the minimum, it offers functionality to register another microservice and its description; list all registered microservices; or fetch a specific object that matches the requested description. This description is the abstraction that both the registry and the microservice depend upon, such that their implementation is decoupled (dependency-inversion-principle).

The **discovery mechanism** scans for available microservices at startup and exposes the registry to them so that they may register themselves. In this way, one can add new microservices without modifying the server (open-closed-principle).

The **bus** exposes server-side objects to the API and to one another. These objects may be microservices from the original registry, but also any number of classes and instances that are created, modified, and discarded during runtime.

### Open API 3.0

The API is defined by an OpenAPI 3.0 compliant specification available at [https://github.com/federatedsecure/api](https://github.com/federatedsecure/api).

For example, the PATCH endpoint reads:

#### Code Listing 5 – Open API 3.0 definition (excerpt)

```yaml
/representation/{representation_uuid}:
    patch:
      summary: call a server-side object
      description: call a server-side object such as a static
      function, a member function, or in case of a class, its
      constructor
      operationId: call_representation
      parameters:
        - in: path
          name: representation_uuid
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        $ref: '#/components/requestBodies/ArgsKwargs'
      responses:
        '200':
          $ref: '#/components/responses/ResponseOk'
        'default':
          $ref: '#/components/responses/ResponseError'
```

### Representation of server-side objects

At the minimum, the bus offers functionality to create representations of registered microservices, create representations of directly uploaded data, create representations of member variables and member functions of represented objects, call a represented function, download the content of a representation, and release a representation. 
Let us have a visual look at the life cycle of server-side object representation:

#### Figure 5

![](images/serverside.png)

At startup, microservices are discovered and announce themselves and their functionality to the registry.
The client requests a microservice with certain abilities, and if a matching microservice is found in the registry, a pointer/handle is stored on the bus, and a UUID of that handle is returned to the client.

In subsequent calls, the client refers to the microservice by that UUID. It may request attributes of that microservice. For example, if the microservice is represented by a class, that attribute may be a member function. Again, a pointer/handle to the attribute is stored on the bus, and another UUID is handed to the client. This process may repeat iteratively.

If a handle represents a callable function, the client may call that function with additional arguments. Again, the result of the function call is not directly returned to the client, but stored on the bus, and yet another UUID is returned to the client. A minimal implementation of this functionality may be illustrated as follows:

#### Code Listing 6 – server-side implementation of calls to server-side objects (simplified)

```python
def call_representation(self, representation_uuid, body):

    args, kwargs = self.get_arguments(body)
    pointer = self.lut_uuid_to_repr[representation_uuid]
    result = pointer(*args, **kwargs)
    
    if result is None:
        return None

    uuid = str(uuid.uuid4())
    self.lut_uuid_to_repr[uuid] = result
    return uuid
```

At some point, the result of the computation is reached. In this case, the client would like to download the result itself instead of a mere representation. The server serializes the result and returns it as a normal response body.

-	**Security consideration:** the client should not be able to access any data on the server. This can be solved by restricting the download functionality to certain objects labeled as output. Proper object-level authorization is thus required in production settings.

Finally, the client may release any representation that it does not need any more. If those representations point to temporarily stored objects, those objects may be deleted. If the representation is of a static microservice or the like, only the representation on the bus is discarded.

-	**Best practice:** The client should send appropriate delete requests to the server whenever client-side representations are discarded or going out of scope. This prevents a buildup of obsolete representations on the bus and memory leakage on the server. As the server cannot control graceful termination of client-side scripts, additional garbage collection mechanisms may be a good idea. For example, automatic removal of unused representations after a certain grace period.

- **Best practice:** The server may keep look-up-tables for the representation of commonly used objects instead of issuing new UUIDs every time.

### Microservices

The architecture is not opinionated on what kind of microservices might be hosted. In the context of Privacy-Preserving Computation (PPC), at least the following types of microservices will most probably be implemented:

-	**(required)** One or more PPC protocols. These microservices will at the minimum provide functionality to build peer-to-peer networks with other servers, accept input data and generate cryptographic shares, and execute the PPC protocol. They may interact with their peers through the bus and the API or through their own third-party networks.

-	**(optional)** Some basic microservices for synchronization, e.g. to broadcast public parameters of calculations to other nodes, or to control the joint flow of computation through semaphores and other signals.

### Webserver and API

The server will expose the public functionality of the bus to the client through an API. A regular webserver will be needed.

-	**Security consideration:** In a production setting, all the usual best practices of securing a webserver and API should be followed. In particular, user authentication and proper object-level authorization.

### Programming language

There is no particular programming language required for the implementation of a Federated Secure Computing server. The propaedeutic reference implementation (see below) is in Python, though.

# Results

## Implementation

### Namespaces

The following namespaces are used or are reserved for future use, see Table 4.

#### Table 4 - Namespaces

<table>
 <thead>
  <tr>
   <td><strong>language</strong></td>
   <td><strong>namespace structure</strong></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td><strong>Python</strong></td>
   <td>
    <ul>
     <li><strong>federatedsecure</strong></li>
     <li>federatedsecure.<strong>client</strong></li>
     <li>federatedsecure.<strong>server</strong></li>
     <li>federatedsecure.<strong>services</strong></li>
     <li>federatedsecure.services.<strong>&#60;name&#62;.*</strong> (see below)</li>
     <li>federatedsecure.services.<strong>&#60;category&#62;.&#60;name&#62;.*</strong> (see below)</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>Java</strong></td>
   <td>
    <ul>
     <li><strong>com.federatedsecure.*</strong></li>
    </ul>
   </td>
  </tr>
 </tbody>
</table>


### Packages

The following packages are available, see Table 5.

#### Table 5 - Packages

<table>
 <thead>
  <tr>
   <td><strong>repository</strong></td>
   <td><strong>packages</strong></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td><strong>PyPI</strong></td>
   <td>
    <ul>
     <li>federatedsecure-client</li>
     <li>federatedsecure-server</li>
     <li>federatedsecure-simon</li>
    </ul>
   </td>
  </tr>
 </tbody>
</table>


### Repository structure

At the time of writing, the following repositories are public on GitHub, see Table 6.

#### Table 6 - GitHub repository structure and contents

<table>
 <thead>
  <tr>
   <td><strong>GitHub repository</strong></td>
   <td><strong>contents</strong></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td><strong>.github</strong></td>
   <td>
    <ul>
     <li>top-level <strong>README.md</strong> of the organization</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>api</strong></td>
   <td>
    <ul>
     <li><strong>OpenAPI 3.0 specification</strong> used by both client and server</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>client-&#60;language&#62;</strong></td>
   <td>
    <ul>
     <li>client libraries providing <strong>API wrappers</strong> in multiple languages</li>
     <li>top-level directory may contain “src”, “test”, “docs” etc.</li>
     <li>examples beyond a simple hello world should go with the services' repositories and should work with multiple clients</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>server</strong></td>
   <td>
    <ul>
     <li><strong>core middleware</strong> as importable library (without webserver runtime)</li>
     <li>implemented in Python only (no top-level language directories)</li>
     <li>top-level directory contains “src”, “docs”, “examples”, and “pypi”</li>
     <li>the “pypi” directory contains the Python Package Index manifest</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>service-name</strong></td>
   <td>
    <ul>
     <li>larger, complex microservices</li>
     <li>typically, <strong>PPC protocols</strong> and interfaces to <strong>3rd party PPC backends</strong></li>
     <li>e.g. “service-simon” (SImple Multiparty computatiON) is a simple, propaedeutic secure multiparty computation (SMPC) protocol </li>
     <li>e.g. “service-datashield” would be an interface to DataSHIELD</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>utility-&#60;category&#62;-&#60;name&#62;</strong></td>
   <td>
    <ul>
     <li>smaller, helper microservices</li>
     <li>e.g. “utility-database-mysql” could expose mysql.connector from the mysqlclient package by wrapping it into a small microservice</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>webserver-&#60;name&#62;</strong></td>
   <td>
    <ul>
     <li>premade <strong>webserver runtimes</strong></li>
     <li>e.g. “webserver-connexion” or “webserver-django”</li>
     <li>further webserver stubs can be generated by the Swagger utilities from the API definition</li>
    </ul>
   </td>
  </tr>
  <tr>
   <td><strong>whitepaper</strong></td>
   <td>
    <ul>
     <li>this whitepaper</li>
    </ul>
   </td>
  </tr>
 </tbody>
</table>


### Correspondence

Namespaces, PyPI packages, and repositories relate as follows, see Table 7.

#### Table 7 - relation between namespaces, packages, and GitHub repositories

<table>
 <thead>
  <tr>
   <td><strong>Python namespace</strong></td>
   <td><strong>PyPI package</strong></td>
   <td><strong>GitHub repository</strong></td>
  </tr>
 </thead>
 <tbody>
  <tr>
   <td>federatedsecure.client</td>
   <td>federatedsecure-client</td>
   <td>client-python</td>
  </tr>
  <tr>
   <td>federatedsecure.server</td>
   <td>federatedsecure-server</td>
   <td>server</td>
  </tr>
  <tr>
   <td>federatedsecure.services.&#60;name&#62;</td>
   <td>federatedsecure-&#60;name&#62;</td>
   <td>service-&#60;name&#62;</td>
  </tr>
  <tr>
   <td>federatedsecure.services.&#60;category&#62;.&#60;name&#62;</td>
   <td>federatedsecure-&#60;category&#62;-&#60;name&#62;</td>
   <td>utility-&#60;category&#62;-&#60;name&#62;</td>
  </tr>
 </tbody>
</table>


### Code availability and licensing

The software is available as free and open source at https://github.com/federatedsecure

All public repositories under this GitHub organization come with the MIT license.

## Installation

### Server-side Installation

First, install a web server to host the API. Flask plus Connexion is a puristic option with minimal overhead. Django is a more complete alternative. Both are provided as premade stubs.

```
git clone https://github.com/federatedsecure/webserver-connexion
cd webserver-connexion
pip install -r requirements.txt
```

Next, install the code server middleware and any additional protocols you might want to use, e.g.:

```
pip install federatedsecure-server
pip install federatedsecure-simon
```

The middleware will automatically discover the plugin, so there is no more additional setup. Run the server by e.g.

```
python ./src/__main__.py --port=55500
```

You can check that the server is running by browsing to

```
curl http://localhost:55500/representations
```

### Client-side Installation

Client-side installation is likely a single command:

```
pip install federatedsecure-client
```

On machines where pip is not readily available as a command line tool (e.g. Android), the following workaround works directly in python:

```
import pip
pip.main([‘install’, ‘federatedsecure-client’])
```

## Benchmarks

### Impact of server hardware

The overwhelming share of computational cost is incurred server-side. In the following benchmark, two respectively three servers and two respectively three clients are simultaneously running on the same localhost machine.

#### Table 8 – speed benchmarks depending on server hardware (seconds)

<table>
 <thead>
  <tr>
   <td>task</td>
   <td>workstation <sup>1</sup></td>
   <td>laptop <sup>2</sup></td>
   <td>Raspberry Zero <sup>3</sup></td>
  </tr>
 </thead>
 <tbody>
  <tr><td colspan="4"><strong>horizontally partitioned data (without record linkage)</strong></td></tr>
  <tr><td>floating point additions <sup>4</sup></td><td>0.10 &#177; 0.01</td><td>0.26 &#177; 0.01</td><td>6.8 &#177; 1.3</td></tr>
  <tr><td>matrix multiplications <sup>6</sup></td><td>0.26 &#177; 0.02</td><td>0.64 &#177; 0.24</td><td>7.7 &#177; 0.2</td></tr>
  <tr><td>histograms <sup>5</sup></td><td>0.25 &#177; 0.04</td><td>0.59 &#177; 0.10</td><td>16.4 &#177; 0.3</td></tr>
  <tr><td>contingency tables <sup>5</sup></td><td>0.38 &#177; 0.07</td><td>1.00 &#177; 0.12</td><td>27.8 &#177; 0.6</td></tr>
  <tr><td>univariate statistics <sup>5</sup></td><td>0.64 &#177; 0.05</td><td>1.71 &#177; 0.18</td><td>52.4 &#177; 0.5</td></tr>
  <tr><td>bivariate statistics <sup>5</sup></td><td>1.93 &#177; 0.05</td><td>5.70 &#177; 0.11</td><td>155.7 &#177; 1.7</td></tr>
  <tr><td>set intersections <sup>5</sup></td><td>0.57 &#177; 0.06</td><td>1.30 &#177; 0.07</td><td>35.7 &#177; 0.5</td></tr>
  <tr><td>set intersection size <sup>5</sup></td><td>0.48 &#177; 0.08</td><td>1.18 &#177; 0.10</td><td>35.9 &#177; 0.7</td></tr>
  <tr><td colspan="4"><strong>vertically partitioned data (with record linkage)</strong></td></tr>
  <tr><td>contingency tables <sup>5</sup></td><td>1.33 &#177; 0.16</td><td>3.29 &#177; 0.35</td><td>84.3 &#177; 2.1</td></tr>
  <tr><td>OLS regression <sup>6</sup></td><td>0.86 &#177; 0.01</td><td>0.24 &#177; 0.01</td><td>5.8 &#177; 0.2</td></tr>
 </tbody>
 <tfoot>
  <tr><td colspan="4"><sup>1</sup> Intel Core i7-9700K, 3.6 GHz, 96 GB DDR4-3600</td></tr>
  <tr><td colspan="4"><sup>2</sup> Intel Core i5-6200U, 2.3 GHz, 8 GB DDR4-2133</td></tr>
  <tr><td colspan="4"><sup>3</sup> Broadcom BCM2835, 1.0 GHz, 512 MB LPDDR2-SDRAM</td></tr>
  <tr><td colspan="4"><sup>4</sup> M=3 parties, N=100 data samples each</td></tr>
  <tr><td colspan="4"><sup>5</sup> M=2 parties, N=100 data samples each</td></tr>
  <tr><td colspan="4"><sup>6</sup> M=2 parties, N=100 elements in 10x10 matrix</td></tr>
 </tfoot>
</table>

The workstation was able to handle the tasks in about a second on average.

The laptop took about two to three times as long. This reflects the lower CPU and RAM clocks, and the fact that it was in energy conservation mode and simultaneously loaded with typical office tasks.

Perhaps most impressively, the Raspberry Zero, a device priced at five US Dollars, is sufficient to run three Federated Secure Computing servers and clients in parallel. The BCM2835 based system-on-a-chip is an order of magnitude slower than the larger machines, but still might be useful in propaedeutic or internet-of-things applications.

### Impact of server-server connectivity

Most secure multiparty computation protocols engage in multiple rounds of communication between the servers, and SIMON (SImple Multiparty computatiON) is no exception. Consequently, the network overhead is expected to have a significant influence on computing time.

The following benchmark connects two servers through different means with varying network latency. The baseline is the workstation as above hosting both servers. The internal latency is way below 1 millisecond and essentially zero. In the second case, the workstation as above and the laptop as above are connected by ethernet cables respectively WLAN through a router with 2 milliseconds latency. In the third case, another fast server is connected to the workstation over public internet with 28 milliseconds of latency:

#### Table 9 – speed benchmarks depending on server-server connectivity (seconds)

<table>
 <thead>
  <tr>
   <td>task</td>
   <td>localhost <sup>1</sup><br>(&lt;1 ms ping)</td>
   <td>LAN/WLAN <sup>2</sup><br>(2 ms ping)</td>
   <td>internet <sup>3</sup><br>(28 ms ping)</td>
  </tr>
 </thead>
 <tbody>
  <tr><td colspan="4"><strong>horizontally partitioned data (without record linkage)</strong></td></tr>
  <tr><td>floating point additions <sup>4</sup></td><td>n/a</td><td>0.50 &#177; 0.02</td><td>2.5 &#177; 0.7</td></tr>
  <tr><td>matrix multiplications <sup>5</sup></td><td>0.26 &#177; 0.02</td><td>0.81 &#177; 0.22</td><td>2.7 &#177; 0.8</td></tr>
  <tr><td>histograms <sup>4</sup></td><td>0.25 &#177; 0.04</td><td>1.72 &#177; 0.05</td><td>9.4 &#177; 2.0</td></tr>
  <tr><td>contingency tables <sup>4</sup></td><td>0.38 &#177; 0.07</td><td>3.12 &#177; 0.13</td><td>16.1 &#177; 4.2</td></tr>
  <tr><td>univariate statistics <sup>4</sup></td><td>0.64 &#177; 0.05</td><td>4.10 &#177; 0.74</td><td>22.3 &#177; 4.5</td></tr>
  <tr><td>bivariate statistics <sup>4</sup></td><td>1.93 &#177; 0.05</td><td>11.14 &#177; 0.54</td><td>59.2 &#177; 5.3</td></tr>
  <tr><td>set intersections <sup>4</sup></td><td>0.57 &#177; 0.06</td><td>1.30 &#177; 0.07</td><td>35.7 &#177; 0.5</td></tr>
  <tr><td>set intersection size <sup>4</sup></td><td>0.48 &#177; 0.08</td><td>1.30 &#177; 0.06</td><td>3.1 &#177; 0.5</td></tr>
  <tr><td colspan="4"><strong>vertically partitioned data (with record linkage)</strong></td></tr>
  <tr><td>contingency tables <sup>4</sup></td><td>1.33 &#177; 0.16</td><td>4.78 &#177; 0.25</td><td>14.9 &#177; 1.9</td></tr>
  <tr><td>OLS regression <sup>5</sup></td><td>0.86 &#177; 0.01</td><td>0.52 &#177; 0.07</td><td>2.3 &#177; 0.4</td></tr>
 </tbody>
 <tfoot>
  <tr><td colspan="4"><sup>1</sup> 1x Intel Core i7-9700K, 3.6 GHz, DDR4-3600, running both servers</td></tr>
  <tr><td colspan="4"><sup>2</sup> 1x Intel Core i7-9700K, 3.6 GHz, DDR4-3600,<br>and 1x Intel Core i5-6200U, 2.3 GHz, DDR4-2133, connected by LAN router, 2 ms RTD</td></tr>
  <tr><td colspan="4"><sup>3</sup> 1x Intel Core i7-9700K, 3.6 GHz, DDR4-3600,<br>and 1x Intel Xeon Silver 4310, 2.1GHz, DDR4-2666; connected by internet, 28±1 ms RTD
</td></tr>
  <tr><td colspan="4"><sup>4</sup> M=2 parties, N=100 data samples each</td></tr>
  <tr><td colspan="4"><sup>5</sup> M=2 parties, N=100 elements in 10x10 matrix</td></tr>
 </tfoot>
</table>

In the WLAN setting, networking overhead about doubles overall computing time, In the internet setting, networking overhead increases computing time about fivefold. Hence, in practical settings, putting servers of the different parties close to each other, e.g. in the same physical data center, or host them on a common cloud infrastructure, will be beneficial.

### Impact of client-server connectivity

In the final benchmark on speed, the servers are run on the same machine as before, but the clients connect through different means.

In the baseline, the clients are run on the same physical machine as above. In the second case, the clients are run on a separate laptop, connected by LAN/WLAN to the workstation as before. In the third setting, the clients are run on smartphones, dialing up to the workstation through public mobile internet services:

#### Table 10 – speed benchmarks depending on client-server connectivity (seconds)

<table>
 <thead>
  <tr>
   <td>task</td>
   <td>localhost <sup>1</sup><br>(&lt;1 ms ping)</td>
   <td>LAN/WLAN <sup>2</sup><br>(2 ms ping)</td>
   <td>internet <sup>3</sup><br>(28 ms ping)</td>
  </tr>
 </thead>
 <tbody>
  <tr><td colspan="4"><strong>horizontally partitioned data (without record linkage)</strong></td></tr>
  <tr><td>floating point additions <sup>4</sup></td><td>0.10 &#177; 0.01</td><td>0.15 &#177; 0.01</td><td>1.07 &#177; 0.21</td></tr>
  <tr><td>matrix multiplications <sup>6</sup></td><td>0.26 &#177; 0.02</td><td>0.32 &#177; 0.02</td><td>1.30 &#177; 0.37</td></tr>
  <tr><td>histograms <sup>5</sup></td><td>0.25 &#177; 0.04</td><td>0.33 &#177; 0.07</td><td>2.90 &#177; 0.59</td></tr>
  <tr><td>contingency tables <sup>5</sup></td><td>0.38 &#177; 0.07</td><td>0.49 &#177; 0.10</td><td>4.59 &#177; 0.97</td></tr>
  <tr><td>univariate statistics <sup>5</sup></td><td>0.64 &#177; 0.05</td><td>69 &#177; 0.05</td><td>8.55 &#177; 1.71</td></tr>
  <tr><td>bivariate statistics <sup>5</sup></td><td>1.93 &#177; 0.05</td><td>2.00 &#177; 0.04</td><td>26.0 &#177; 3.34</td></tr>
  <tr><td>set intersections <sup>5</sup></td><td>0.57 &#177; 0.06</td><td>0.59 &#177; 0.08</td><td>1.91 &#177; 0.17</td></tr>
  <tr><td>set intersection size <sup>5</sup></td><td>0.48 &#177; 0.08</td><td>0.51 &#177; 0.14</td><td>1.62 &#177; 0.20</td></tr>
  <tr><td colspan="4"><strong>vertically partitioned data (with record linkage)</strong></td></tr>
  <tr><td>contingency tables <sup>5</sup></td><td>1.33 &#177; 0.16</td><td>1.54 &#177; 0.09</td><td>4.91 &#177; 0.91</td></tr>
  <tr><td>OLS regression <sup>6</sup></td><td>0.86 &#177; 0.01</td><td>0.15 &#177; 0.02</td><td>1.74 &#177; 0.28</td></tr>
 </tbody>
 <tfoot>
  <tr><td colspan="4"><sup>1</sup> servers and clients are running in separate CPU cores on same machine</td></tr>
  <tr><td colspan="4"><sup>2</sup> clients on laptop connected through Intel Dualband-Wireless-AC 8260</td></tr>
  <tr><td colspan="4"><sup>3</sup> clients on three separate mobile phones (2x Samsung SM-G52F/DS “XCover 5”)<br>connecting through 4G network of Telefónica S.A. in Germany
</td></tr>
  <tr><td colspan="4"><sup>4</sup> M=3 parties, N=100 data samples each</td></tr>
  <tr><td colspan="4"><sup>5</sup> M=2 parties, N=100 data samples each</td></tr>
  <tr><td colspan="4"><sup>6</sup> M=2 parties, N=100 elements in 10x10 matrix</td></tr>
 </tfoot>
</table>

Clients connected through localhost and LAN/WLAN client were about as fast, but connections over the mobile network were slower by an order of magnitude due to increased round trip delays.

#### Figure 6 - Scripting and running a client on an Android smartphone

![](images/figure6a.png) ![](images/figure6b.png)

### Code size benchmarks

Both the client-side API wrapper and the server-side middleware stub are small:

#### Table 11 – size of client-side API wrapper

<table>
 <thead>
  <tr>
   <td>language</td>
   <td>without HTTP interface</td>
   <td>with HTTP interface</td>
  </tr>
 </thead>
 <tbody>
  <tr><td>Python</td><td>2.8 kilobyte</td><td>7.4 kilobyte</td></tr>
  <tr><td>R</td><td>2.1 kilobyte</td><td>6.6 kilobyte</td></tr>
  <tr><td>Javascript</td><td>2.1 kilobyte</td><td>3.4 kilobyte</td></tr>
 </tbody>
</table>

#### Table 12 – size of server-side API wrapper

<table>
 <thead>
  <tr>
   <td>language</td>
   <td>without webserver</td>
   <td>with webserver</td>
  </tr>
 </thead>
 <tbody>
   <td>Python</td><td>12.4 kilobyte</td><td>31.6 kilobyte</td>
 </tbody>
</table>
