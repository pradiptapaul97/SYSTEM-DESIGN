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
