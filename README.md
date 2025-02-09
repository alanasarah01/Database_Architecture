# 📦 Amazonas - E-commerce Database Architecture  

This project implements the **database architecture for a scalable e-commerce** using **MongoDB Atlas on Azure**. The database model was designed in **Hackolade**, and data was inserted via **MongoDB Compass**.  

The database architecture was designed to support **exponential growth, high availability, and query efficiency**, leveraging **sharding, replicas, and NoSQL best practices**.  

---

## 📌 Technologies Used  
- **MongoDB Atlas** - NoSQL database hosted on Azure  
- **MongoDB Compass** - GUI tool for database management  
- **Hackolade** - NoSQL database modeling tool  
- **Azure (California, USA region)** - Chosen for better performance, scalability, and cost-efficiency  

---

## 📌 Database Architecture  
The database schema was modeled for **high scalability and performance**, combining **denormalized structures and references** between collections to optimize queries.  

### **Collections and Relationships**  
| **Collection** | **Related to** | **Foreign Key** | **Description** |
|--------------|--------------|----------------|----------------|
| `customer` | `address` | `address.address_id` | Stores customer data and references their addresses |
| `address` | `customer` | `customer.customer_id` | Keeps addresses in a separate collection to avoid redundancy |
| `order` | `customer`, `product`, `payment`, `delivery` | `customer.customer_id`, `product.product_id`, `payment.payment_id`, `delivery.delivery_id` | Records customer orders, keeping historical prices and status |
| `product` | `review` | `review.review_id` | Contains products available for sale, referencing their reviews |
| `review` | `product`, `customer` | `product.product_id`, `customer.customer_id` | Keeps reviews separate to optimize performance |
| `delivery` | `order` | `order.oder_id` | Tracks delivery status separately |
| `payment` | `order`, `customer` | `order.oder_id`, `customer.customer_id` | Logs payments independently for auditing purposes |
| `cart` | `customer`, `product` | `customer.customer_id`, `product.product_id` | Allows frequent updates without impacting orders |

---

## 📌 Sharding and Replication Strategy  

### **Sharding (Data Partitioning)**
To support high transaction volumes and concurrent access, the database is designed for **horizontal sharding**. The following collections will be sharded:  
1. **`order`** → Sharded by `customer_id` (ensures that orders from the same customer are stored together)  
2. **`product`** → Sharded by `category` (optimizes category-based product searches)  
3. **`customer`** → Sharded by `customer_id` using hashing (distributes customers evenly across shards)  

Additionally, **chunks** will be used to divide shards and ensure **automatic load balancing**.

### **Replication for High Availability**
The MongoDB Atlas database is already configured with a **3-node Replica Set** (1 **Primary** and 2 **Secondaries**).  

- **Automatic failover**: If the Primary node fails, a Secondary node is automatically promoted.  
- **Distributed reading**: Reports and historical queries can be directed to Secondary nodes (`secondaryPreferred`).  

---

## 📌 Strategies for Growth and High Concurrency  

1️⃣ **Lifecycle-Based Storage**  
- Old orders are periodically archived to reduce the load on the main database.  
- Logs and historical data are moved to optimized storage.  

2️⃣ **Continuous Monitoring on Azure**  
- Monitoring tools are used to identify bottlenecks and adjust infrastructure as needed.  

3️⃣ **Smart Indexing**  
- Indexes are created for frequently queried fields (`email`, `order_id`, `category`, `product_id`).  
- Composite indexes are used for specific queries, such as retrieving **recent orders by customer**:
  ```javascript
  db.order.createIndex({ "customer_id": 1, "date": -1 })
