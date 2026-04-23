# System Design

## Single Server Setup Data Flow
<span style="color: red;">**Note:** Ideal for small user bases, struggles for heavy traffic.</span>

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
