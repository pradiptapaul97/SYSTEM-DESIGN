# System Design

## Single Server Setup Data Flow
**Note:** Ideal for small user bases, struggles for heavy traffic.

![alt text](<Screenshot (9).png>)

A single server setup is the most basic architecture where all components (web server, application, and database) run on a single machine. Here is how the data communication works:

1. **Website Name (Domain Name):** The user enters a human-readable website name (e.g., `www.example.com`) into their web browser.
2. **DNS Server (Domain Name System):** The browser doesn't know how to reach `www.example.com` directly. It queries a DNS Server, which acts like the internet's phonebook. The DNS server translates the human-readable domain name into an IP address.
3. **IP Address (Internet Protocol):** The DNS server responds with the IP address (e.g., `192.168.1.100`) associated with the domain. This IP address represents the exact location of the single server on the internet.
4. **Data Communication (Request & Response):**
   - **Request:** Now that the browser has the IP address, it sends an HTTP/HTTPS request directly to the single server over the internet.
   - **Processing:** The single server receives the request, processes it (which may involve reading from its local database or executing application logic), and prepares the data.
   - **Response:** Finally, the server sends back an HTTP/HTTPS response containing the requested resources (HTML, CSS, JavaScript, images) to the user's browser, and the website is rendered on the screen.

## Web Tier and Data Tier Separation

![alt text](<Screenshot (10).png>)

As the number of users increases, a single server will eventually run out of resources (CPU, RAM, Storage) and struggle to handle the heavy traffic. To resolve this, the single server architecture is divided into two distinct tiers:

1. **Web Tier (Web/Application Server):**
   - **Role:** This tier is responsible for handling incoming HTTP requests from the users' browsers, running the application's business logic, and serving web pages or API responses.
   - **Advantage:** By isolating the web server, you can efficiently handle more concurrent user requests. If traffic spikes, you can scale this tier independently without worrying about the database.

2. **Data Tier (Database Server):**
   - **Role:** This tier is strictly dedicated to storing, retrieving, and managing the application's data. It does not handle direct internet traffic.
   - **Advantage:** Database operations are often resource-heavy. Giving the database its own dedicated server prevents complex queries from slowing down the web server. It also improves security since the data tier can be placed in a private network, inaccessible directly from the internet.

### Key Benefits of this Separation:
- **Independent Scaling:** You can upgrade or add more web servers (for traffic) or upgrade the database server (for storage/compute) individually based on what is becoming the bottleneck.
- **Better Performance:** Each server uses its dedicated resources exclusively for its specific task, preventing them from competing for CPU or memory.
- **Improved Security:** The database is no longer directly exposed to the public internet; it only communicates with the trusted Web Tier.

## Choosing the Right Type of Database
![alt text](<Screenshot (11).png>)

When designing your Data Tier, selecting the appropriate database architecture is a critical decision. There are **two main options**:

### 1. Relational Databases (RDBMS)
Relational databases are highly structured and organize data into predefined **tables and rows** with strict relationships between them.

- **Key Characteristics:** Ensures strong data consistency and is ideal for structured data and complex queries.
- **Query Language:** They use **SQL (Structured Query Language)** for finding and manipulating data.
- **Examples:** PostgreSQL, MySQL, SQLite, Oracle Database.

#### Advantages of RDBMS:
- **Complex Queries:** They support complex **JOIN operations** across multiple tables, making it easy to fetch related data.
- **Data Consistency & Integrity:** They provide strict data integrity, especially important for transactions. Each transaction reliably follows the **ACID** properties:
  - **A (Atomicity):** Ensures that a transaction is treated as a single, indivisible unit. Either all operations within it succeed, or none do (all-or-nothing).
  - **C (Consistency):** Ensures the database transitions from one valid state to another. Any data written must follow the defined rules and constraints.
  - **I (Isolation):** Ensures that concurrent transactions execute independently without interfering with each other. The result is the same as if they were executed sequentially.
  - **D (Durability):** Ensures that once a transaction is committed, it remains permanently stored, even in the event of a system failure or crash.

### 2. Non-Relational Databases (NoSQL)
NoSQL databases provide much more flexibility, as they do not require a fixed schema. They are heavily used to store, manage, and quickly access large amounts of **unstructured or semi-structured data**.

- **Key Characteristics:** Highly scalable, flexible data models, and excellent for rapid development.
- **Different Forms of NoSQL:**
  
  - **Document Stores:** Store data in JSON-like documents. Example: **MongoDB**
  ![alt text](<Screenshot (12).png>)
    ```mermaid
    classDiagram
        class Document1 {
            _id: "101"
            name: "Alice"
            age: "28"
            city: "NY"
        }
        class Document2 {
            _id: "102"
            name: "Bob"
            hobbies: "Reading, Gaming"
        }
    ```
    - **Advantages:** High schema flexibility, easy mapping to application objects, excellent for hierarchical data.
    - **Disadvantages:** Poor performance on complex joins across multiple documents, can lead to data duplication.

  - **Wide-Column Stores:** Store data in tables, rows, and dynamic columns. Example: **Cassandra/Cosmos DB**
  ![alt text](<Screenshot (13).png>)
    ```mermaid
    classDiagram
        class RowKey_User1 {
            name: "Alice"
            email: "alice@web.com"
        }
        class RowKey_User2 {
            name: "Bob"
            age: "32"
        }
    ```
    - **Advantages:** Extreme horizontal scalability, extremely fast write performance, built for high availability and big data.
    - **Disadvantages:** Poor at querying by anything other than the primary key, complex data modeling, not suited for complex aggregations.

  - **Key-Value Stores:** Store data as a collection of key-value pairs. Example: **Redis/Memcached**
  ![alt text](<Screenshot (16).png>)
    ```mermaid
    graph LR
        K1["Key: 'session:101'"] --> V1["Value: '{user: 1, active: true}'"]
        K2["Key: 'cart:55'"] --> V2["Value: '{item: laptop, qty: 1}'"]
    ```
    - **Advantages:** Blazing fast read/write speeds, very simple data model, highly scalable for caching and session management.
    - **Disadvantages:** Cannot query by the "value", lack of complex query capabilities, not designed for complex relationships.

  - **Graph Databases:** Store data in nodes and edges, focusing on relationships. Example: **Neo4j**
  ![alt text](<Screenshot (15).png>)
    ```mermaid
    graph LR
        A((Alice)) -- KNOWS --> B((Bob))
        A -- LIVES_IN --> C((New York))
        B -- WORKS_AT --> D((Tech Corp))
        D -- LOCATED_IN --> C
    ```
    - **Advantages:** Perfectly suited for highly interconnected data (social networks, fraud detection, recommendation engines), lightning-fast relationship traversals.
    - **Disadvantages:** Steeper learning curve (requires query languages like Cypher), harder to scale horizontally compared to other NoSQL databases, overkill for simple tabular data.

### Summary Comparison of NoSQL Databases

| NoSQL Type | Data Model | Key Strength | Main Limitation | Best Use Case (When to use) |
| :--- | :--- | :--- | :--- | :--- |
| **Document Store** | JSON/BSON Documents | Highly flexible schema, maps easily to code objects | Poor at complex joins and multi-document transactions | Content management, e-commerce catalogs, user profiles |
| **Wide-Column Store** | Tables with dynamic columns | Extreme write performance and horizontal scalability | Difficult to query by non-primary keys | Time-series data, IoT sensor data, massive logging systems |
| **Key-Value Store** | Key-Value pairs | Blazing fast read/write speeds, highly scalable | Cannot query by value, very limited query language | Caching (e.g., user sessions), leaderboards, real-time recommendations |
| **Graph Database** | Nodes and Edges | Lightning-fast complex relationship traversals | Harder to scale horizontally, steep learning curve | Social networks, fraud detection, recommendation engines |

---

## When to Choose Relational vs. Non-Relational

### Choose a Relational Database (RDBMS) when:
1. **Well-Structured Data & Clear Relationships:** Your data is highly structured and entities have strict relationships.
   - *Example:* An e-commerce app tracking customers and orders.
2. **Strong Consistency & Transactional Integrity:** You require strict data integrity and cannot afford any anomalies (ACID compliance).
   - *Example:* A financial application or banking system.

### Choose a Non-Relational Database (NoSQL) when:
1. **Super Low Latency:** You need incredibly rapid, quick responses for read and write operations.
2. **Unstructured & Semi-Structured Data:** Your data does not fit into rigid tables and schemas frequently change.
3. **Massive Data Volumes:** You require highly scalable storage capable of handling massive amounts of traffic and data across distributed servers.

---

## Scaling the System: Vertical vs. Horizontal Scaling

When your application starts receiving heavy traffic and your current server setup can no longer handle the load, you need to scale. There are two primary ways to scale a system: **Vertical Scaling** and **Horizontal Scaling**.

### 1. Vertical Scaling (Scale-Up)
- **Understanding the Content:** Vertical scaling means adding more power (CPU, RAM, Storage, etc.) to your existing server. You are essentially making your single machine stronger.
- **Example:** Upgrading your server from 8GB of RAM and 4 CPUs to 64GB of RAM and 16 CPUs.

```mermaid
flowchart LR
    S1["Server<br>(8GB RAM, 4 CPU)"] -- "Scale Up" --> S2["Server<br>(64GB RAM, 16 CPU)"]
    
    style S1 fill:#f9f,stroke:#333,stroke-width:2px
    style S2 fill:#bbf,stroke:#333,stroke-width:4px
```

**Advantages:**
- Very simple to implement (usually no code changes required).
- Less complex administration and maintenance.
- Data consistency is naturally maintained since everything is in one place.

**Disadvantages:**
- **Hardware Limits:** There is a hard physical limit to how much you can upgrade a single machine.
- **Single Point of Failure:** If the server goes down, the entire application goes offline.
- **Downtime:** Upgrading hardware often requires taking the server offline temporarily.

### 2. Horizontal Scaling (Scale-Out)
- **Understanding the Content:** Horizontal scaling means adding more servers into your pool of resources. Instead of making one server stronger, you add more servers to distribute the load using a Load Balancer.
- **Example:** Going from running your application on 1 server to running it simultaneously on 10 identical servers.

```mermaid
flowchart LR
    S1["Single App Server"] -- "Scale Out" --> LB{"Load Balancer"}
    LB --> S2["App Server 1"]
    LB --> S3["App Server 2"]
    LB --> S4["App Server 3"]
    
    style LB fill:#f90,stroke:#333,stroke-width:2px
    style S1 fill:#bbf,stroke:#333,stroke-width:2px
    style S2 fill:#bbf,stroke:#333,stroke-width:2px
    style S3 fill:#bbf,stroke:#333,stroke-width:2px
    style S4 fill:#bbf,stroke:#333,stroke-width:2px
```

**Advantages:**
- **Infinite Scalability:** You can theoretically keep adding an endless number of servers.
- **High Availability & Fault Tolerance:** If one server crashes, the others can take over, preventing system downtime.
- **No Downtime Scaling:** You can add or remove servers dynamically without taking the system offline.

**Disadvantages:**
- Highly complex to implement and manage.
- Requires software architecture changes (e.g., making applications stateless, implementing distributed caching).
- Data consistency becomes much harder to maintain across multiple servers.

### Summary Comparison Table

| Feature | Vertical Scaling (Scale-Up) | Horizontal Scaling (Scale-Out) |
| :--- | :--- | :--- |
| **Definition** | Adding more resources (CPU/RAM) to an existing server | Adding more servers to the existing resource pool |
| **Complexity** | Simple | Highly Complex |
| **Limits** | Hard hardware limits (cannot scale infinitely) | Practically infinite scalability |
| **Single Point of Failure**| Yes (If the server dies, the app dies) | No (Built-in redundancy and high availability) |
| **Downtime** | Often requires downtime to upgrade hardware | Zero downtime (servers can be added dynamically) |
| **Cost** | High-end hardware can be very expensive | Uses cheaper, standard commodity hardware |

### When to Use Which? (Scenarios)

#### Use Vertical Scaling when:
- **Small to Medium Applications:** You have a small engineering team and want the easiest way to handle moderate growth quickly without rewriting code.
- **Traditional Relational Databases:** SQL databases (RDBMS) are notoriously difficult to scale horizontally, so they are typically scaled vertically first.

#### Use Horizontal Scaling when:
- **Large-Scale Applications:** You anticipate massive traffic that no single machine could ever handle (e.g., global social media or e-commerce platforms).
- **Stateless Web Services:** If your web/application servers don't store local user session data, they can easily be scaled horizontally to handle sudden traffic spikes dynamically.

---

## Load Balancers

A **Load Balancer** distributes incoming network traffic across multiple servers. This ensures that no single server bears too much load, which improves overall application responsiveness and availability.

### 7 Strategies and Algorithms Used in Load Balancing:

#### 1. Round Robin
- **Description:** The simplest algorithm. It routes each incoming request to the next server in the line, cycling through the list of servers in order.
- **Workflow:** Request 1 goes to Server A, Request 2 goes to Server B, Request 3 goes to Server C, Request 4 goes back to Server A, and so on.
- **Advantages:** Extremely simple to implement; ensures a mathematically even distribution of requests.
- **Disadvantages:** Does not account for server health, current load, or capacity. Can easily overload a server that gets stuck processing heavy requests.

#### 2. Least Connection
- **Description:** Routes traffic to the server with the fewest active connections at the time the request is received.
- **Workflow:** When a new request arrives, the load balancer checks the active connection count for all servers. If Server A has 50 connections and Server B has 10, the load balancer sends the new request to Server B.
- **Advantages:** Highly efficient at balancing actual load, preventing individual servers from getting overwhelmed.
- **Disadvantages:** Requires the load balancer to constantly compute and track connection states, adding slight overhead.

#### 3. Least Response Time
- **Description:** Directs traffic to the server with the fewest active connections *and* the lowest average response time.
- **Workflow:** The load balancer monitors how quickly each server responds. It combines this speed metric with the active connection count to find the currently "fastest" and "most available" server, routing the next request there.
- **Advantages:** Excellent for ensuring fast user experiences; automatically shifts traffic away from slow or struggling servers.
- **Disadvantages:** More complex to compute; temporary network hiccups can heavily skew response time metrics.

#### 4. IP Hash
- **Description:** Uses a mathematical function (hashing) on the client's IP address to determine which server receives the request.
- **Workflow:** The user's IP (e.g., `192.168.1.5`) is hashed into a number. That number dictates the server choice. Because the hash function is consistent, the same IP address will *always* be routed to the same server (great for sticky sessions).
- **Advantages:** Guarantees session persistence (sticky sessions) because a specific user always hits the same server.
- **Disadvantages:** Can lead to uneven load distribution if a large group of users originates from the same IP network (e.g., a corporate office).

#### 5. Weighted Algorithm (Weighted Round Robin)
- **Description:** Allows administrators to assign a "weight" (priority or capacity value) to each server based on its hardware capabilities.
- **Workflow:** If Server A is twice as powerful as Server B, it is given a Weight of 2. The load balancer will then send two requests to Server A for every one request it sends to Server B.
- **Advantages:** Maximizes resource utilization by intentionally sending more traffic to more powerful hardware.
- **Disadvantages:** Requires manual configuration and constant tweaking of weights if server specs change.

#### 6. Geographic Routing Algorithm
- **Description:** Distributes requests based on the physical, geographical location of the user making the request. *(Note: Often what is meant by "graphical" routing).*
- **Workflow:** A user in Tokyo makes a request. The load balancer detects the location via their IP address and routes the request to the data center physically located in Japan, rather than sending it to a server in New York.
- **Advantages:** Drastically reduces network latency for global users and helps comply with strict regional data residency laws.
- **Disadvantages:** Highly complex to set up; relies heavily on accurate DNS resolution and GeoIP databases.

#### 7. Consistent Hashing
- **Description:** An advanced hashing technique that minimizes the redistribution of traffic when servers are added or removed from the cluster.
- **Workflow:** Servers and incoming requests are mapped onto a circular "hash ring". A request is routed to the first server it encounters moving clockwise on the ring. If a server goes down, only the traffic meant for that specific server gets reassigned to the next one, leaving the rest of the network's traffic untouched.
- **Advantages:** Extremely resilient to scaling events; adding or removing nodes only affects a tiny fraction of user sessions.
- **Disadvantages:** Harder to implement properly; not strictly necessary unless operating at massive scale or dealing with distributed caching.

### Summary Comparison of Load Balancing Algorithms

| Algorithm | Key Characteristic | Main Advantage | Main Disadvantage |
| :--- | :--- | :--- | :--- |
| **Round Robin** | Cycles requests sequentially | Simple to implement | Ignores server load and health |
| **Least Connection** | Routes to fewest active connections | Prevents server overload | Requires connection tracking overhead |
| **Least Response Time** | Routes to fastest responding server | Optimizes for user speed | Complex to calculate accurately |
| **IP Hash** | Hashes client IP to assign server | Ensures session persistence | Can cause uneven load distribution |
| **Weighted Algorithm** | Prioritizes servers based on specs | Maximizes powerful hardware | Requires manual weight configuration |
| **Geographic Routing** | Routes based on physical location | Minimizes global network latency | Requires complex GeoIP setup |
| **Consistent Hashing** | Uses a hash ring for assignments | Seamless scaling with minimal disruption | Complex to implement, overkill for small setups |

---

### Popular Load Balancing Solutions

| Category | Solution | Key Features / Notes |
| :--- | :--- | :--- |
| **Software** | **Nginx** | High performance, versatile; includes built-in **health checks** to monitor server status. |
| | **HAProxy** | Reliable, high-performance TCP/HTTP load balancer; widely used for its efficiency. |
| **Hardware** | **F5 BIG-IP** | Enterprise-grade hardware load balancer with advanced security and traffic management. |
| | **Citrix ADC** | Scalable application delivery controller (formerly NetScaler) for high-availability environments. |
| **Cloud** | **AWS Elastic Load Balancing (ELB)** | Fully managed load balancing for AWS environments (ALB, NLB, GLB). |
| | **Azure Load Balancer** | Layer-4 load balancer for high availability in Microsoft Azure. |
| | **Google Cloud Load Balancing** | Scalable, software-defined load balancing for Google Cloud Platform. |

---

### Single point of Failure

**Single Point of Failure (SPOF)** refers to a component, process, or device in a system that, if it fails, will cause the entire system or application to fail. In a high-availability or fault-tolerant system, the goal is to eliminate SPOFs to ensure continuous operation even if individual components fail.

![alt text](<Screenshot (27).png>)

#### Critical Examples of SPOFs in Distributed Systems

While architectural scaling improves performance, it can inadvertently introduce new bottlenecks if high availability is not baked into every tier.

##### 1. The Monolithic Database (Data Tier SPOF)
In many early-stage architectures, while the Web Tier is scaled horizontally, the **Data Tier** remains a single, centralized instance.
- **The Technical Risk:** A single database server represents a catastrophic point of failure. Whether it's a hardware crash, a targeted SQL-based attack, or a resource-exhaustion DDoS, if the database becomes unresponsive, the system's "source of truth" disappears.
- **System Impact:** Even with dozens of healthy application servers, the system cannot process requests that require data persistence or retrieval. This results in a **Total System Outage** where users see generic error pages.
- **Professional Mitigation:** Implement **Database Replication** (Primary-Replica or Multi-Primary) and automated **Failover mechanisms**. This ensures that if the primary node fails, a standby node can immediately take over traffic.

##### 2. The Traffic Orchestrator (Load Balancer SPOF)
A Load Balancer is designed to protect the system from individual server failures, but if only one Load Balancer is deployed, it becomes the ultimate bottleneck.
- **The Technical Risk:** If the Load Balancer is the sole entry point for all incoming traffic, it is a high-value target. A hardware failure, network misconfiguration, or a massive DDoS attack on the Load Balancer's IP will sever all connections between your users and your infrastructure.
- **System Impact:** This creates a "Black Hole" effect. Despite having a fully functional backend of 100+ servers, your application becomes **unreachable**, making it appear as if the entire platform is down.
- **Professional Mitigation:** Deploy **Redundant Load Balancers** in an **Active-Passive** or **Active-Active** configuration. Utilizing **Floating IPs (Virtual IPs)** or DNS-based Global Server Load Balancing (GSLB) ensures that if one orchestrator fails, traffic is seamlessly rerouted through a healthy peer.

---

### Strategies for Eliminating Single Points of Failure (SPOFs)

To build a truly resilient system, architects must move beyond simple scaling and implement comprehensive **High Availability (HA)** strategies. Below are the standard industry patterns for ensuring system uptime.

#### 1. High Availability through Redundancy
![alt text](<Screenshot (29).png>)
The foundation of a fault-tolerant system is the elimination of "lonely" components. This is achieved by deploying multiple instances of critical infrastructure.
- **Active-Passive Configuration:** A primary load balancer handles all traffic, while a secondary "standby" node remains idle. If the primary fails, a **Virtual IP (VIP)** or **Floating IP** is instantly remapped to the secondary node using protocols like **Keepalived** or **VRRP**.
- **Active-Active Configuration:** Multiple load balancers operate simultaneously, sharing the total traffic load. If one node fails, the remaining nodes automatically absorb its traffic, ensuring zero downtime and optimized resource utilization.

#### 2. Comprehensive Health Monitoring
![alt text](<Screenshot (28).png>)
Monitoring must extend beyond the application layer to include the infrastructure orchestrators themselves.
- **External Heartbeats:** Use external monitoring services (e.g., AWS CloudWatch, Datadog, or Prometheus) to perform constant "heartbeat" checks on the load balancers and the network path.
- **Automated Failover Triggers:** When a monitoring agent detects that a load balancer is unresponsive or failing health checks, it can automatically trigger a DNS record update or a network routing change to bypass the faulty node.

#### 3. Self-Healing Infrastructure
![alt text](<Screenshot (32).png>)
Modern distributed systems prioritize **Resilience by Design** through automated recovery and orchestration.
- **Auto-Scaling & Auto-Provisioning:** By utilizing cloud-native services like **AWS Auto Scaling Groups** or **Kubernetes Controllers**, the system can automatically detect the loss of a node.
- **Automated Instance Replacement:** Instead of requiring manual intervention, the platform terminates the unhealthy instance and immediately provisions a fresh, pre-configured load balancer from a standard image (AMI or Container). This ensures the system "heals" itself back to its target state within seconds, minimizing the window of vulnerability.

---

## API (Application Programming Interface)

An **API (Application Programming Interface)** is a set of defined rules and protocols that allow different software applications to communicate and exchange data with each other. In modern system design, APIs serve as the "connective tissue" between decoupled services, enabling them to work together as a unified platform.

### What is an API? (The Technical Definition)
At its core, an API acts as a formal contract between a **provider** (the server) and a **consumer** (the client). It abstracts the underlying complexity of the system, allowing developers to interact with services without needing to understand their internal code or database structure.

#### The Request-Response Cycle
Communication through an API typically follows a structured cycle:
1. **Request:** The client sends a structured message to an endpoint (e.g., `GET /v1/users/101`).
2. **Processing:** The server validates the request, checks for authorization, executes business logic, and retrieves data from the Data Tier.
3. **Response:** The server returns an HTTP status code (e.g., `200 OK`) along with the requested payload, usually formatted as **JSON** or **XML**.

---

### Core Components of an API Interaction
To design or consume APIs effectively, developers must master these four primary components:

| Component | Description | Example |
| :--- | :--- | :--- |
| **Endpoint** | The specific URL or address where the API resource resides. | `https://api.myapp.com/v1/orders` |
| **HTTP Method** | The verb that defines the type of action to be performed. | `GET` (Read), `POST` (Create), `DELETE` (Remove) |
| **Headers** | Metadata providing context about the request or the client. | `Authorization: Bearer <token>`, `Content-Type: application/json` |
| **Body (Payload)** | The actual data sent to the server (usually for creation or updates). | `{"product_id": 55, "quantity": 2}` |

---

### Popular API Architectures

![alt text](<Screenshot (33).png>)

#### 1. REST (Representational State Transfer)
The most widely used architecture for web services. It is based on standard HTTP protocols and emphasizes **statelessness**, where each request contains all the information needed to fulfill it.

##### Understanding "Resource-Based" Design
In REST, everything is treated as a **Resource**. A resource is any object, data, or service that can be accessed by a client (e.g., a User, an Order, or a Product).

- **The URI as an Identity:** Every resource is identified by a unique **URI (Uniform Resource Identifier)**. Instead of calling a function like `getUser(id)`, you access a path that represents the user entity.
- **Nouns vs. Verbs:** RESTful URIs should always use **nouns** to represent resources, never verbs. The action to be performed is determined by the **HTTP Method** (the verb), not the URL.

| Action | Professional REST (Resource-Based) | Unprofessional (Action-Based) |
| :--- | :--- | :--- |
| **Get all users** | `GET /users` | `GET /getAllUsers` |
| **Get a specific user** | `GET /users/123` | `GET /getUser?id=123` |
| **Create a user** | `POST /users` | `POST /createUser` |
| **Delete a user** | `DELETE /users/123` | `POST /deleteUser/123` |

##### Collections and Sub-Resources
Resources can be grouped into collections or nested to show relationships:
- **Collection:** `/products` refers to the entire list of products.
- **Individual Resource:** `/products/55` refers to a specific product.
- **Sub-Resource:** `/users/123/orders` refers to all orders belonging to user 123.

##### Understanding "Statelessness"
In a **Stateless** architecture, the server does not store any information about the client's state or previous interactions. Each request from a client to a server must be "self-contained."

- **Self-Contained Requests:** Every single request must include all the information the server needs to process it—including authentication tokens, parameters, and the desired action. The server doesn't "remember" that you logged in during the previous request.
- **Authentication via Tokens:** Because the server is stateless, it doesn't use traditional server-side sessions. Instead, clients typically send an **Authorization Header** (e.g., a **JWT** or **Bearer Token**) with every single request to prove their identity.
- **Impact on Scalability:** Statelessness is a key reason why REST APIs are so scalable. Since any server in a pool can handle any request (because no session data is stored locally), you can add or remove servers behind a load balancer without ever worrying about "session stickiness" or synchronizing user data across nodes.



#### 3. gRPC (Google Remote Procedure Call)
A high-performance framework that uses **Protocol Buffers** (a binary serialization format) instead of JSON. It is primarily used for lightning-fast communication between microservices.

##### Understanding Protocol Buffers (Protobuf)
Protocol Buffers are gRPC's **Interface Definition Language (IDL)**. They define how data is structured and how services interact.

- **Binary Serialization:** Unlike REST (which uses text-based JSON), Protobuf is a **binary format**. This makes the data significantly smaller and much faster to serialize/deserialize, resulting in lower latency and reduced bandwidth usage.
- **Strongly Typed & Schema-First:** You must define your data structure in a `.proto` file before writing any code. This ensures that both the client and server agree on the data format, reducing runtime errors.

##### Example of a `.proto` Definition
This file defines a service and the structure of the messages it exchanges:

```protobuf
syntax = "proto3"; // Using the latest version of Protobuf

// Define the service and its methods
service UserService {
  // A simple RPC: Client sends a request, Server returns a response
  rpc GetUser (UserRequest) returns (UserResponse);
}

// Define the request message
message UserRequest {
  string user_id = 1; // The number '1' is a unique tag for binary mapping
}

// Define the response message
message UserResponse {
  string name = 1;
  string email = 2;
  int32 age = 3;
}
```

##### Why use gRPC over REST?
- **Speed:** Binary format is 5x to 10x faster than JSON.
- **Streaming:** Supports bidirectional streaming (client and server can send a stream of messages simultaneously).
- **Code Generation:** Protobuf automatically generates client and server code in multiple languages (Java, Python, Go, Node.js, etc.) based on the `.proto` file.

#### 4. Webhooks
Often called "Reverse APIs," webhooks allow a server to push real-time data to a client automatically when a specific event occurs (e.g., a payment is completed), rather than the client constantly polling the server for updates.

![alt text](<Screenshot (34).png>)

![alt text](<Screenshot (36).png>)

---

## The API Design Process: A 4-Step Framework

Before writing a single line of code or defining an endpoint, an architect must think through the system's requirements. This structured approach ensures the API is functional, scalable, and secure.

### Step 1: Identify Core Use Cases & User Stories
**The Goal:** Understand exactly what the users (and other systems) are trying to achieve.

- **The Question:** *"What are the specific actions a user needs to perform?"*
- **E-commerce Example:**
    - User can browse products.
    - User can add items to a cart.
    - Admin can manage inventory levels.
- **API Choice Insight:**
    - **REST:** Ideal for standard CRUD operations (Products, Orders).
    - **GraphQL:** Best if the frontend requires flexible, varying data structures.
    - **gRPC:** Best for internal service orchestration (Order → Payment → Inventory).

### Step 2: Define Scope & Boundaries
**The Goal:** Establish clear responsibilities for each service and endpoint (Microservices approach).

- **The Question:** *"What does this API handle, and what is outside its responsibility?"*
- **Example:**
    - **Product Service:** Handles `/products` ONLY.
    - **Order Service:** Handles `/orders` ONLY.
- **API Choice Insight:**
    - **REST:** Provides clean, predictable boundaries for public APIs.
    - **gRPC:** The industry standard for service-to-service communication within the same boundary.
    - **GraphQL:** Can serve as a **BFF (Backend-for-Frontend)** layer, aggregating data from multiple services.

### Step 3: Determine Performance Requirements
**The Goal:** Select the right communication protocol based on speed and real-time needs.

- **The Question:** *"How many concurrent users are expected, and how fast must the response be?"*
- **Comparison of Performance Profiles:**
    - **Social Media Feed:** Needs high flexibility + fast loading → **GraphQL** (reduces round trips).
    - **Inter-Service Payments:** Needs extreme low latency + reliability → **gRPC** (binary serialization).
    - **Live Stock Prices:** Needs real-time streaming → **WebSockets**.
    - **Simple Admin Dashboard:** Simple CRUD without strict latency needs → **REST** (cache-friendly).

### Step 4: Consider Security Constraints
**The Goal:** Ensure the data is protected and access is restricted based on roles.

- **The Question:** *"Who can access this data, and how do we prevent abuse (like DDoS or data leaks)?"*
- **Architectural Security Patterns:**
    - **REST/GraphQL:** Typically secured via **JWT (JSON Web Tokens)** or **OAuth2** in HTTP headers.
    - **gRPC:** Often uses **mTLS (Mutual TLS)** for highly secure service-to-service authentication.
    - **WebSockets:** Requires token validation during the initial "handshake" before a long-lived connection is established.

---

### Real-World Case Study: Food Delivery App

| Phase | Strategy | API Selection |
| :--- | :--- | :--- |
| **Step 1: Use Cases** | Browse menus, place orders, track drivers in real-time. | Mixed Stack |
| **Step 2: Boundaries** | Separate services for Restaurants, Orders, and Delivery. | Microservices |
| **Step 3: Performance** | Use gRPC for backend calls; WebSockets for live driver location. | **gRPC & WebSockets** |
| **Step 4: Security** | Role-based access for Users, Drivers, and Partners. | **JWT & HTTPS** |

---

### 🧠 The Architectural Mental Model

- **Use Cases** → Defines **WHAT** the API does.
- **Scope** → Defines **WHERE** the boundaries are.
- **Performance** → Defines **HOW FAST** it responds.
- **Security** → Defines **HOW SAFE** the data remains.

**Final Takeaway for Developers:**
A strong modern architecture often uses **all styles together**:
- **Frontend** connects to a **GraphQL** gateway for flexible queries.
- **Gateway** connects to **REST** APIs for business logic.
- **Microservices** communicate internally via **gRPC** for maximum speed.
- **Real-time features** (Chat, Tracking) use **WebSockets**.

---

## API Design Methodologies

How you initiate the design of an API significantly impacts the system's stability, scalability, and developer experience. There are three primary methodologies used in professional backend development.

### 1. Top-Down Approach (Product-Driven)
**The Philosophy:** Start from business requirements and user experience (UX), then design the API to support those needs.

- **The Flow:** Business Requirements → API Design → Implementation → Database Schema.
- **Example:** In a food delivery app, you first identify that a user needs to "View Restaurants." You then design `GET /restaurants` and finally build the database tables to support that endpoint.
- **Pros:**
    - Clean, intuitive API design.
    - Perfectly aligned with frontend/client needs.
- **Cons:**
    - May lead to complex database queries if the schema isn't optimized for the API design.
- **Best For:** Public-facing REST APIs and Frontend-driven GraphQL implementations.

### 2. Bottom-Up Approach (Data-Driven)
**The Philosophy:** Start with the existing database or internal services and expose them as API endpoints.

- **The Flow:** Database Schema → Internal Services → API Layer → User.
- **Example:** You already have a `payments` table, so you simply create a `GET /payments` endpoint that returns all columns from that table.
- **Pros:**
    - Extremely fast to implement.
    - Ideal for internal utility tools.
- **Cons:**
    - Often results in poor API design (exposing internal DB structures).
    - Hard to evolve without breaking the frontend.
- **Best For:** Internal Microservices (gRPC) and rapid prototyping.

### 3. Contract-First Approach (Architecture-Driven)
**The Philosophy:** Start by defining a formal agreement (the contract) between the frontend and backend teams before writing any implementation code.

- **The Flow:** API Contract (Spec) → Parallel Implementation (Frontend & Backend).
- **Example:** Using **OpenAPI (Swagger)** for REST or **.proto files** for gRPC to define all endpoints, request bodies, and response types upfront.
- **Pros:**
    - Enables frontend and backend teams to work in parallel.
    - Ensures high consistency and documentation-first development.
- **Cons:**
    - Requires significant upfront planning and discipline.
- **Best For:** Large-scale systems, cross-team collaborations, and mission-critical services.

---

### Comparison of Design Methodologies

| Methodology | Starting Point | Primary Focus | Ideal Use Case |
| :--- | :--- | :--- | :--- |
| **Top-Down** | User Needs | User Experience | Public APIs / Web Apps |
| **Bottom-Up** | Database | Data Exposure | Internal Tools / Prototypes |
| **Contract-First** | API Spec | System Architecture | Large-Scale / Enterprise Systems |

---

### 🚀 Real-World Execution: The Hybrid Workflow
Mature engineering teams often combine these three approaches into a high-performance hybrid workflow:
1. **Top-Down:** Identify the core user stories and business logic.
2. **Contract-First:** Define the formal API specification (Swagger/Proto) so all teams stay aligned.
3. **Bottom-Up:** Implement the backend services and optimize the database for the defined contract.

---

### 🧠 Developer Intuition
- If you think like a **Product Engineer** → You prefer **Top-Down**.
- If you think like a **Database Engineer** → You prefer **Bottom-Up**.
- If you think like a **System Architect** → You prefer **Contract-First**.

---

## API Communication Protocols (Real-time APIs)

![alt text](<Screenshot (46)-1.png>)

![alt text](<Screenshot (47).png>)

![alt text](<Screenshot (48).png>)

![alt text](<Screenshot (49).png>)

![alt text](<Screenshot (50).png>)

![alt text](<Screenshot (53).png>)

![alt text](<Screenshot (54).png>)

---

## Transport Layer Protocols: TCP vs. UDP

The **Transport Layer** is responsible for end-to-end communication and data transfer between applications. In system design, choosing between **TCP** and **UDP** depends entirely on whether your application prioritizes **Reliability** or **Speed**.

### 1. TCP (Transmission Control Protocol)
**The Philosophy:** "Slow but Steady." TCP ensures that every packet sent is received correctly and in the exact order it was sent.

- **Connection-Oriented:** Before any data is sent, TCP establishes a formal connection between the sender and receiver using a **3-Way Handshake**.
- **Reliable & Ordered:** If a packet is lost, TCP automatically detects it and retransmits it. It also reassembles packets in the correct order if they arrive out of sequence.
- **Flow Control & Congestion Control:** TCP slows down the transmission if the network is congested or if the receiver is overwhelmed.

> [!TIP]
> **Real-World Analogy:** Think of TCP like sending a package via a courier with **proper tracking, delivery receipts, and a required signature**. If the package is lost, the courier finds it or sends a replacement.


#### The TCP 3-Way Handshake
```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: SYN (Let's connect!)
    Server-->>Client: SYN-ACK (I'm ready, let's go!)
    Client->>Server: ACK (Great! Connection Established)
```

- **Best For:** Applications where data integrity is critical. (e.g., Web Browsing, Email, Database queries).

---

### 2. UDP (User Datagram Protocol)
**The Philosophy:** "Fast and Furious." UDP sends data packets (datagrams) as quickly as possible without checking if they ever arrived.

- **Connectionless:** There is no handshake. Data is simply "fired" at the receiver's IP address.
- **Unreliable & Unordered:** If a packet is lost, it is gone forever. Packets may also arrive in any order.
- **Low Overhead:** Because it doesn't track connections or retransmit data, UDP is significantly faster and uses less bandwidth than TCP.

#### The UDP Workflow
```mermaid
flowchart LR
    Sender((Sender)) -- "Packet 1" --> Receiver((Receiver))
    Sender -- "Packet 2" --> Receiver
    Sender -. "Packet 3 (LOST)" .-> X[X]
    Sender -- "Packet 4" --> Receiver
```

- **Best For:** Applications where speed and low latency are more important than perfect data (e.g., Live Streaming, VoIP, Online Gaming).

---

### Comparison: TCP vs. UDP

| Feature | TCP (Reliable) | UDP (Fast) |
| :--- | :--- | :--- |
| **Connection** | Connection-Oriented (3-Way Handshake) | Connectionless (No Handshake) |
| **Reliability** | **Guaranteed Delivery** (Retransmits lost data) | **No Guarantee** (Lost data is gone) |
| **Ordering** | Delivers data in the exact order sent | Data can arrive in any order |
| **Speed** | Slower (due to overhead and checking) | **Blazing Fast** (minimal overhead) |
| **Header Size** | Large (20 - 60 Bytes) | Small (8 Bytes) |
| **Flow Control** | Yes (Prevents overwhelming the receiver) | No (Fire and forget) |

### 🚀 When to Use Which? (System Design Scenarios)

#### Use TCP when you need **Data Integrity**:
- **HTTP / HTTPS:** Browsing the web requires every image and text block to load correctly.
- **SSH / FTP:** Remote access and file transfers cannot afford corrupted or missing files.
- **SMTP (Email):** You want to ensure the entire message reaches the recipient.

#### Use UDP when you need **Real-time Speed**:
- **Online Gaming:** If you lose a "packet" representing a player's movement, you don't want the game to freeze while it's retransmitted; you just want the next movement update.
- **Video Conferencing / VoIP:** A tiny bit of "lag" or "glitch" in audio is better than the entire call pausing to buffer a lost packet.
- **DNS (Domain Name System):** Requests are small and need to be fast. If a DNS request fails, the browser just tries again.

---

## Handling Large Datasets: Filtering, Sorting, and Pagination
When an API returns a collection (e.g., thousands of products), returning all items at once causes high latency, heavy server load, and slow client rendering.

### 1. Filtering & Sorting
- **Filtering:** Allows clients to request only a subset of data that meets specific criteria.
  - *Example:* `GET /products?category=laptop&brand=apple`
- **Sorting:** Allows clients to define the order of the results.
  - *Example:* `GET /products?sort=price_asc` (Price low to high) or `GET /products?sort=-created_at` (Newest first)

### 2. Pagination (Two Primary Methods)

#### A. Offset-based Pagination
This is the most common method, using a fixed "starting point" and a "limit."
- **Example:** `GET /items?offset=20&limit=10` (Skip the first 20 items, give me the next 10).
- **Pros:** 
  - Simple to implement (Direct SQL `OFFSET` and `LIMIT`).
  - Allows "Page Jumping" (Users can click directly on "Page 5").
- **Cons:** 
  - **Performance Issues:** As the offset increases (e.g., offset 1,000,000), the database still has to scan all previous rows before skipping them.
  - **Data Drift:** If a new item is added while a user is on Page 1, the last item of Page 1 might show up again as the first item of Page 2.

#### B. Cursor-based Pagination (Keyset Pagination)
Instead of skipping a number of rows, you use a "pointer" (cursor) to the last item seen.
- **Example:** `GET /items?cursor=item_123&limit=10` (Give me 10 items starting *after* item_123).
- **Pros:**
  - **High Performance:** The database can use an index to jump directly to the cursor, making it efficient for massive datasets.
  - **Consistent Results:** Not affected by new data being added/deleted; items don't "shift" between pages.
- **Cons:**
  - **No Page Jumping:** You cannot skip directly to "Page 10"; you must fetch Page 1, then Page 2, etc.
  - **Complex Implementation:** Requires unique, sortable columns (like timestamps or UUIDs) as cursors.

---

| Feature | Offset-based | Cursor-based |
| :--- | :--- | :--- |
| **Best For** | Small datasets, Admin tables | **Large datasets, Infinite scrolls** |
| **UX Style** | Standard "Page 1, 2, 3..." | "Load More" or Infinite Scroll |
| **Performance** | Slower as you go deeper | **Consistently Fast** |
| **Implementation** | Very Easy | Complex |

### 🚀 Where to Use Which?
- **Use Offset-based** for internal tools, admin dashboards, or lists where "Total Pages" and "Jumping to a page" are required.
- **Use Cursor-based** for social media feeds (Twitter/Instagram), real-time logs, or any high-volume public-facing list where performance is key.

![alt text](<Screenshot (56).png>)

![alt text](<Screenshot (57).png>)

---

## GraphQL: What is it and Why use it?
**The Philosophy:** "Ask for exactly what you need, and nothing more." GraphQL is a query language for APIs and a runtime for fulfilling those queries with your existing data.

### Why Use GraphQL? (The Core Benefits)

1. **Eliminating Over-fetching & Under-fetching:**
   - **Over-fetching:** REST often returns a massive JSON object with fields the client doesn't need (wasting bandwidth). GraphQL allows the client to specify only the fields it intends to use.
   - **Under-fetching:** REST might not return enough data, forcing the client to make multiple requests (increasing latency). GraphQL allows nested resource fetching in a single call.

2. **Single Network Round Trip:**
   In REST, fetching a user and their last 5 orders might require multiple round trips. With **GraphQL**, you consolidate these into one query:
   ```graphql
   query {
     user(id: "1") {
       name
       orders(last: 5) {
         id
         total_price
       }
     }
   }
   ```

3. **Strongly Typed Schema:**
   GraphQL uses a **Schema Definition Language (SDL)** to define the types of data available. This acts as a contract between the frontend and backend.

4. **Single Endpoint:**
   Unlike REST (which has many endpoints like `/users`, `/products`), GraphQL typically uses a **single `/graphql` endpoint**.

### Core Components of GraphQL
- **Queries:** Used for **fetching** data (equivalent to `GET` in REST).
- **Mutations:** Used for **modifying** data (equivalent to `POST`, `PUT`, `DELETE`).
- **Subscriptions:** Used for **real-time** updates via WebSockets.

---

### Comparison: REST vs. GraphQL

| Feature | REST | GraphQL |
| :--- | :--- | :--- |
| **Data Fetching** | Multiple endpoints, fixed data structure | Single endpoint, flexible data structure |
| **Efficiency** | Prone to Over/Under-fetching | **No Over/Under-fetching** |
| **Versioning** | Harder (requires `/v1/`, `/v2/`) | **Versionless** (just add/deprecate fields) |
| **Caching** | Built-in HTTP caching (Easy) | Complex (requires custom client-side caching) |
| **Learning Curve** | Low (uses standard HTTP) | High (requires learning query syntax/schema) |

### 🚀 When to Use GraphQL?
- **Complex Apps:** When your frontend requires deeply nested data from multiple sources.
- **Mobile Apps:** When bandwidth is limited and you want to minimize payload size.
- **Microservices:** When you want a "Gateway" (BFF) to aggregate data from multiple services.

### Authentication (AuthN)
**The Philosophy:** "Who are you?" Authentication is the process of verifying that a user or system is who they claim to be.

![alt text](<Screenshot (62).png>)

#### Authentication vs. Authorization (The Key Difference)
In system design, these two terms are often confused but serve very different purposes.

| Feature | Authentication (AuthN) | Authorization (AuthZ) |
| :--- | :--- | :--- |
| **Question** | Who are you? | What are you allowed to do? |
| **Goal** | Verify Identity | Verify Permissions |
| **Example** | Logging in with a password. | Checking if a user is an "Admin" before deleting a file. |
| **Occurs...** | **First** (Identity must be known) | **Second** (Permissions are checked after login) |


### Authentication Methods

![alt text](<Screenshot (63).png>)

### Basic Authentication Methods

#### 1. Basic Authentication
The client sends the username and password in the HTTP `Authorization` header, encoded as **Base64**.

**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: GET /resource (Authorization: Basic dXNlcjpwYXNz)
    Server->>Server: Base64 Decode & Verify DB
    Server-->>Client: 200 OK (Resource Data)
```

- **Pros:** Extremely simple to implement; supported by all browsers.
- **Cons:** **Insecure** (Credentials are sent in plain-text Base64); no session logout mechanism.
- **Test Cases:**
    - [ ] **TC-1:** Verify access with correct `username:password` (Expect 200 OK).
    - [ ] **TC-2:** Verify access with incorrect credentials (Expect 401 Unauthorized).
    - [ ] **TC-3:** Verify access with missing Authorization header (Expect 401 Unauthorized).

---

#### 2. Digest Authentication
A more secure alternative to Basic Auth that uses a **challenge-response** mechanism to avoid sending passwords in plain text.

**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: GET /resource
    Server-->>Client: 401 Unauthorized (WWW-Authenticate: Digest nonce="abc...")
    Note over Client: Hash(User + Pass + Nonce)
    Client->>Server: GET /resource (Authorization: Digest response="hash...")
    Server-->>Client: 200 OK
```

- **Pros:** Password is never sent over the wire; more resilient to replay attacks (using nonces).
- **Cons:** Vulnerable to Man-in-the-Middle if not using HTTPS; harder to implement than Basic Auth.
- **Test Cases:**
    - [ ] **TC-1:** Verify server sends `401` with `nonce` on first request.
    - [ ] **TC-2:** Verify client generates correct MD5 hash and gets `200 OK`.
    - [ ] **TC-3:** Verify request fails if `nonce` is stale or reused.

---

#### 3. API Keys
A long-lived unique string (identifier) assigned to a user or service, typically passed in a header or query parameter.

![alt text](<Screenshot (64).png>)

**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: GET /api/v1/data (X-API-KEY: my-secret-key)
    Server->>Server: Look up Key in Database/Cache
    Server-->>Client: 200 OK
```

- **Pros:** Simple for machine-to-machine communication; easy to rate-limit per key.
- **Cons:** If leaked, the key provides full access until manually revoked; keys are often stored in plain text by clients.
- **Test Cases:**
    - [ ] **TC-1:** Verify valid API Key allows access.
    - [ ] **TC-2:** Verify invalid/deleted API Key returns `403 Forbidden`.
    - [ ] **TC-3:** Verify rate-limiting works when a key exceeds its quota.

---

#### 4. Session-based Authentication
The server creates a session for the user after login and stores it (usually in a DB or Redis), sending a **Session ID** back to the client via a cookie.

![alt text](<Screenshot (66).png>)

**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant Redis
    Client->>Server: POST /login (user/pass)
    Server->>Redis: Store Session {id: 123, user: 'alice'}
    Server-->>Client: 200 OK (Set-Cookie: session_id=123)
    Note over Client: Subsequent Request
    Client->>Server: GET /dashboard (Cookie: session_id=123)
    Server->>Redis: Get Session 123
    Server-->>Client: 200 OK
```

- **Pros:** Secure (sensitive data stays on server); sessions can be easily revoked (e.g., "Logout from all devices").
- **Cons:** Stateful (hurts horizontal scaling); vulnerable to CSRF (Cross-Site Request Forgery).
- **Test Cases:**
    - [ ] **TC-1:** Verify Session ID is generated and stored upon login.
    - [ ] **TC-2:** Verify accessing protected route without a cookie returns `401`.
    - [ ] **TC-3:** Verify session is deleted from the store upon logout.

---

### Comparison Table: 4 Types of Authentication

| Feature | Basic Auth | Digest Auth | API Keys | Session-based |
| :--- | :--- | :--- | :--- | :--- |
| **Storage** | None (Send every time) | None (Send every time) | Persistent Key | **Server-side (Stateful)** |
| **Security** | Low (Plaintext-like) | Medium (Hashed) | Medium (Secret Key) | **High (Server-controlled)** |
| **Complexity**| Very Low | Medium | Low | High |
| **Scalability** | High | High | High | Low (Requires Shared Store) |
| **Best For** | Internal testing | Legacy systems | Public APIs | **Web Applications** |


### Token Based Authentication Methods

Token-based authentication is **stateless**, meaning the server does not need to store session data. The "token" itself contains all the information needed to identify the user.

#### 1. JWT (JSON Web Tokens) & Bearer Tokens
A **Bearer Token** is a security token that gives access to the "bearer" (the person who holds it). **JWT** is the most popular format for these tokens.

> [!NOTE]
> **Bearer vs. JWT: Don't get confused!**
> - **Bearer Token:** Is the **how**. It defines the *transport mechanism* (e.g., sending the token in the `Authorization: Bearer <token>` header). It's like the **Envelope**.
> - **JWT:** Is the **what**. It defines the *data format* of the token itself. It's like the **Letter** inside the envelope.
> *Note: A Bearer token doesn't HAVE to be a JWT (it could be a simple string), but most modern APIs use JWTs as Bearer tokens.*


**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant Client
    participant Server
    Client->>Server: POST /login (user/pass)
    Server->>Server: Verify & Create JWT (Signed)
    Server-->>Client: 200 OK (token: "header.payload.sig")
    Note over Client: Subsequent Request
    Client->>Server: GET /data (Authorization: Bearer <JWT>)
    Server->>Server: Verify Signature & Expiry
    Server-->>Client: 200 OK
```

- **JWT Structure:** 
    - `Header`: Algorithm & Token type.
    - `Payload`: User data (Claims) like `user_id` and `exp` (expiry).
    - `Signature`: Ensures the token hasn't been tampered with.
- **Pros:** **Stateless & Scalable** (No DB lookup needed); works perfectly for Microservices.
- **Cons:** Hard to revoke (Tokens are valid until they expire); Large header size if payload is big.
- **Test Cases:**
    - [ ] **TC-1:** Verify valid JWT grants access.
    - [ ] **TC-2:** Verify tampered JWT (modified payload) is rejected (Expect 401).
    - [ ] **TC-3:** Verify expired JWT is rejected.

---

#### 2. Access & Refresh Tokens
To improve security, we use **Short-lived Access Tokens** (e.g., 15 mins) and **Long-lived Refresh Tokens** (e.g., 7 days).

![alt text](<Screenshot (67).png>)

**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant Client
    participant Server
    participant DB
    Client->>Server: POST /login
    Server-->>Client: {access_token: "short", refresh_token: "long"}
    Note over Client: Access Token Expires...
    Client->>Server: GET /resource (access_token)
    Server-->>Client: 401 Unauthorized (Expired)
    Client->>Server: POST /refresh (refresh_token)
    Server->>DB: Is refresh_token valid?
    DB-->>Server: Yes
    Server-->>Client: {new_access_token: "short"}
```

- **Pros:** **Better Security** (If an access token is stolen, it's only useful for a few minutes); Refresh tokens can be revoked in the DB to "log out" a user.
- **Cons:** More complex to implement on the frontend (Requires logic to handle token refreshing silently).
- **Test Cases:**
    - [ ] **TC-1:** Verify Access Token works for its duration.
    - [ ] **TC-2:** Verify Refresh Token can generate a new Access Token after expiry.
    - [ ] **TC-3:** Verify that revoking a Refresh Token prevents further access.

---

### Comparison: JWT vs. Access/Refresh Tokens

| Feature | Simple JWT (Bearer) | Access & Refresh Tokens |
| :--- | :--- | :--- |
| **Lifecycle** | Single token with moderate life | **Two tokens (Short & Long lived)** |
| **Security** | Medium (Wait for expiry) | **High (Rotation & Revocation)** |
| **Complexity** | Low | **High** |
| **Best For** | Simple APIs / Internal tools | **Secure Web & Mobile Apps** |

### OAuth2 and OIDC Authentication Framework

![alt text](<Screenshot (68).png>)

![alt text](<Screenshot (69).png>)

**OAuth 2.0** is an **authorization** framework that allows an application to access resources on behalf of a user without seeing their password. **OpenID Connect (OIDC)** is a simple **identity** layer on top of OAuth 2.0 that allows the app to verify the identity of the user.

- **OAuth2 (Authorization):** "Can I access your Google Photos for you?" (Valet Key).
- **OIDC (Authentication):** "Who are you exactly?" (ID Card).

#### Core Roles in OAuth2/OIDC:
1. **Resource Owner:** The User (e.g., You).
2. **Client:** The Application (e.g., Canva).
3. **Authorization Server:** The Identity Provider (e.g., Google, GitHub).
4. **Resource Server:** The API containing the data (e.g., Google Photos API).

---

#### The Authorization Code Flow (Most Secure)
This is the standard flow for most web and mobile applications.

**Graphical Workflow:**
```mermaid
sequenceDiagram
    participant User
    participant Client as App (Canva)
    participant AuthServer as Auth Server (Google)
    
    User->>Client: Click "Login with Google"
    Client->>User: Redirect to Google Login
    User->>AuthServer: Enter Credentials & Consent
    AuthServer-->>Client: Send Authorization Code
    Client->>AuthServer: Exchange Code for Tokens (+ Client Secret)
    AuthServer-->>Client: Issue ID Token (OIDC) & Access Token (OAuth2)
    Client->>User: Login Successful
```

- **Pros:** **Highly Secure** (Credentials never touch the app); users don't need to create new accounts (SSO).
- **Cons:** Very complex to implement from scratch; requires multiple network round trips.
- **Test Cases:**
    - [ ] **TC-1:** Verify redirection to the correct Identity Provider (IDP) URL.
    - [ ] **TC-2:** Verify `code` is exchanged for tokens correctly.
    - [ ] **TC-3:** Verify `ID Token` contains correct user claims (email, name).
    - [ ] **TC-4:** Verify that an invalid or reused `code` is rejected.

---

### Comparison: OAuth2 vs. OIDC

| Feature | OAuth 2.0 | OIDC (OpenID Connect) |
| :--- | :--- | :--- |
| **Primary Goal** | **Authorization** (Access) | **Authentication** (Identity) |
| **Token Issued** | **Access Token** | **ID Token** (and Access Token) |
| **Token Format** | Any (often opaque or JWT) | **Strictly JWT** |
| **Information** | Permissions (Scopes) | User Profile (Claims like email, sub) |
| **Analogy** | A key to a specific room | A passport verifying who you are |

### 🚀 When to Use?
- **Use OAuth2** when you want one app to access data in another app (e.g., Buffer posting to your Twitter).
- **Use OIDC** when you want a "Login with X" button to handle user registration and login.


