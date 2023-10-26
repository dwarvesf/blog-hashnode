---
title: "Overcoming Distributed System Challenges Using Golang"
seoTitle: "Overcoming Distributed System Challenges Using Golang"
seoDescription: "Golang helps simplify distributed systems with efficient concurrency, libraries, and tools. Handles session management, communication, and scalability."
datePublished: Fri May 26 2023 09:46:25 GMT+0000 (Coordinated Universal Time)
cuid: cli4dot8n000f0ajmb2po837w
slug: overcoming-distributed-system-challenges-using-golang
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1685094160159/0b61f28a-466f-4736-b03e-68307b48afbc.png
tags: challenge, go, concurrency, distributed-system, blockchain

---

*At Dwarves, we set monthly themes for our company to research an area of programming. We wanted to focus on Golang and some of the more common problems and nuances companies face, such as distributed systems. Below are our findings of some solutions to developing distributed systems on Go.*

## Introduction

Most systems today are developed based on cloud computing and distributed architecture to achieve high efficiency and cost optimization, such as email systems, social networks, e-commerce platforms, etc. They can be deployed in multiple regions and connected through proprietary protocols to ensure data safety and continuity. In addition, with a distributed architecture, systems can improve the number of users at a given time, users in different regions can have better access speeds, avoiding local errors affecting the entire system, expanding the system by region, upgrading and testing for each user group.

Golang is a programming language developed for building distributed systems and is increasingly being used and converted by many companies. Golang is designed to handle concurrent tasks, capable of handling thousands of concurrent connections. Go's performance is as good as C in many tasks and faster than other languages like javascript, python, Ruby, etc. Some famous distributed systems written in Golang are Ethereum blockchain, docker swarm, etc.

## Objective

The aim of this article is to analyze the advantages and disadvantages of distributed systems, the challenges they pose, and potential solutions. We will apply these concepts to a real-world problem we have encountered, and propose new ideas for future projects.

We will introduce technologies and tools used to develop distributed systems, such as distributed databases, distributed computing, and Golang-based distributed algorithms. However, we will also discuss the difficulties of using Golang to build distributed systems.

Fortunately, Golang offers many libraries to support building distributed systems, such as "etcd", a distributed system used to store configuration information; "consul", a distributed system used to manage services and connection information; and "go-micro", a framework for building distributed applications with support for gRPC and RESTful APIs. Furthermore, Golang supports popular protocols like HTTP, TCP, and UDP, as well as WebSockets and gRPC for interaction between nodes in the distributed system.

## Challenges & Solutions

### **Handling failure modes in hard real-time distributed systems**

The difficulty lies in debugging Golang in a distributed environment. When developing a distributed system, finding and fixing bugs is challenging due to the dynamic nature of the system. This requires a comprehensive testing and debugging strategy to ensure the reliability and maintenance of the distributed system.

There is no definitive solution to this problem. To ensure the distributed system operates well, we need to focus on testing the system and writing unit tests for as many possible scenarios as possible.

### **Concurrency challenges**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685094237106/91fa412b-9898-45a0-89c0-b3ad65f534b7.png align="center")

#### **Problem**

This situation occurs when two or more tasks access and modify the same data simultaneously, called a **race condition**. The end result may be inconsistent or incorrect.

**Deadlocks** can also occur when two or more tasks wait for each other's resources and cannot continue to execute. At that point, the system will be stuck and will not work.

**Data inconsistency** also occurs when data is not synchronized properly and may be inconsistent between different nodes.

#### **Solution**

Increasing the **scalability** of the distributed system also means increasing the complexity of concurrent processing. Therefore, the distributed system needs to be designed to ensure good concurrent processing.

**Load balancing** is one of the methods that help to minimize high loads in the distributed system. However, concurrent processing in different nodes also poses challenges in evenly distributing the load across the nodes.

Managing **distributed locks** in a distributed system is also one of the challenges for concurrent processing. Nodes must synchronize access to locks to avoid race conditions or deadlocks.

In addition, Golang has native concurrency features based on go routines and channels that we can take advantage of managing data types for distributed use cases.

### **Communication challenges**

#### **Problem**

One of the challenges is **session management**. When using the HTTP protocol, every time a new connection is created, the system will create a new session and need to manage that session to ensure data integrity and access rights.

Another challenge in Golang communication is **error handling**. When sending requests from one component to another in a distributed system, errors can occur during transmission.

In addition, there are other communication challenges in Golang in distributed systems such as **state management, data synchronization, and ensuring data integrity** across different nodes.

#### **Solution**

Using distributed communication libraries is a possible solution to apply to these projects. Golang provides many communication libraries such as RPC, HTTP, WebSocket, MQTT, NATS, and many other protocols. Using these libraries can help manage communication more effectively and minimize errors.

Designing the architecture of a distributed system to ensure data integrity and efficient session management. Components in the system need to be designed to communicate with each other easily and effectively.

Handle errors and state management more carefully. Errors that occur during transmission need to be handled flexibly and mechanisms such as sending requests again or using backup versions can be used to ensure data integrity.

Use data synchronization mechanisms to ensure data integrity across different nodes in the distributed system. Distributed lock or distributed transaction mechanisms can be used for data synchronization.

As well as apply appropriate design patterns such as Circuit Breaker or Retry Pattern to manage distributed communication effectively and ensure system readiness.

### **Distributed database challenges**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685094249496/d3d4de51-40d4-4f20-834f-285d1cfebd50.png align="center")

#### **Problem**

Synchronizing distributed data across multiple nodes makes data synchronization more complex. When one node changes data, it must ensure that other nodes are also updated with the latest data.

In addition, ensuring consistency with a distributed database, ensuring data consistency is an important issue. This requires nodes to synchronize data with each other to avoid any nodes containing different data from other nodes.

Furthermore, handling conflicts when multiple nodes access the same data at the same time, conflicts can occur. Locks and synchronization are tools to solve this problem.

Another challenge is improving performance when multiple nodes access the same data at the same time. Optimizing performance becomes more important than ever in distributed systems, especially when you have contention between data access and stateless flows.

#### **Solution**

One solution is to use a distributed database management system. Database management systems such as Cassandra, MongoDB, or Apache HBase are designed to operate on multiple nodes and support synchronization and high performance. Golang supports these databases, and it also has frameworks like Beego or Revel that support developing distributed web applications.

Using sharding techniques to divide data into smaller parts that are stored on different nodes can also help. This technique helps minimize the load on each node and increase performance.

Using data synchronization tools such as ZooKeeper could help for consensus on data access and likewise for using blockchain to store distributed data.

### **Security challenges**

Managing database access in a distributed system poses significant challenges, as data is stored on multiple nodes. Ensuring secure and efficient database access across nodes becomes complex. Additionally, user authentication and authorization become more difficult when spanning multiple nodes, requiring careful management and authorization processes. Securing distributed data on each node requires close coordination. Network security is crucial to safeguard the system, implementing access control, monitoring, and detection mechanisms to counter network attacks. Distributed systems are attractive targets for attackers, making protection against threats like DDoS, SQL injection, and cross-site scripting imperative.

To address these challenges, several solutions can be implemented. Encryption tools such as TLS, SSL, AES, and SHA can be utilized to enhance data security. Designing a robust authentication and authorization architecture ensures that only authorized users or modules can access critical resources. Network security solutions like firewalls and virtual private networks can be employed to defend against external attacks and fortify system defenses. Implementing comprehensive monitoring and intrusion detection systems can help detect and respond to any potential security breaches promptly. By combining these solutions, the distributed system can mitigate risks and provide a secure environment for data and operations.

### **Scalability challenges**

#### **Problem**

We can scale Golang through horizontal scaling to meet growth needs, likely through containers. However, scaling a Golang distributed system may encounter difficulties, especially in managing database access and synchronizing data. The more nodes in the system, the higher the latency due to geographic distance in transmitting data between nodes causing delays in the system.

#### **Solution**

Using load balancing and caching solutions can help to minimize latency. In addition, using a Consensus Protocol can help ensure the resolution of conflicts between nodes. We can also help to ensure consistent versions between nodes through data modeling.

## Case study p**roject iCrosschain**

iCrossChain is a promising blockchain bridge project with strong decentralized system architecture, providing fast processing speed, high security, and availability. If implemented and used correctly, iCrossChain could become one of the most advanced blockchain bridge projects today.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685094262157/cec4e719-a36f-4c57-90bc-9539cc538ff8.png align="center")

### Challenges

The system needs to support multiple users using it at the same time with queries to the blockchain network causing congestion and returning incorrect results at the time of receiving requests. The system also needs to ensure the safety of customers' transaction processing requests and ensure that transactions are verified accurately.

We also need to expand support for different blockchains without increasing the verification delay of transactions. Expanding on that, we will eventually have to deal with the issue of concurrent transaction processing when multiple processes are trying to access the same resource.

Finally, there will be a considerate amount of difficulty managing resources to ensure the system is capable of managing memory, and network bandwidth to efficiently use resources.

### **Solutions**

**Architecture**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1685094269927/73248919-b3cd-4ee3-b507-91d8cd55f560.png align="center")

*iCrosschain system*

The system is designed with a core computing component that allows for horizontal scaling, meaning that when there are many requests from clients, the system will run additional instances on new infrastructure to serve requests from client.

In addition, this system builds proxies to allow connections to multiple blockchains at the same time and reduce access limitations. The system also builds upn independent validators to verify transactions, data is synchronized from the blockchain to avoid transaction forgery and system deception.

Using go routines for each blockchain index, minimizing errors on each blockchain that do not affect other chains.

#### **Database design**

* **PostgreSQL:** We use PostgreSQL as the main database for the system due to its scalability and high security, which allows us to easily upgrade when necessary.
    
    **Memory caching:** We use memory caching for some static information to reduce the number of queries to the database as well as reduce response time for the client.
    

#### **Technical & Tools**

The following are the main technologies and tools used in the iCrosschain system.

* **Terraform**: An infrastructure as code (IaC) configuration encoding tool used to manage software infrastructure. It allows you to build, change, and manage software infrastructure reliably and easily using configuration encoding instead of manually operating infrastructure components.
    
* **Kubernetes**: A distributed container management platform used to deploy and manage distributed applications. Kubernetes provides features such as automatic scaling, automatic recovery, and service update capabilities while still keeping the application running continuously.
    
* **Grafana Loki**: An open-source application used to display and monitor data. It allows users to connect to various data sources such as databases, logs, and applications to create and view dashboards, alerts, and notifications from all instances and modules.
    
* **Load balancer**: Amazon Load Balancer addresses access routing and load distribution issues for web applications and APIs. ALB provides load balancing features based on content, source IP address, tags, and many other unique features.
    

### **Evaluation**

During our testing, we evaluated a few key areas for this system. We specifically targeted large-scale load tests, autoscale testing, and introducing a validator to the system:

* **Large-scale load testing**: In this test, we attempted to push a large number of requests into the system to check its stability. After the test, we identified some areas for improvement, such as the minimum memory requirement for each instance and when to launch a new instance.
    
* **Automatic system scale-up and scale-down testing**: In this test, we attempted to put the system in a situation with a large number of simultaneous accesses to trigger the system to automatically scale up by adding new instances until it can meet demand. After the test, we identified some scaling milestones, such as a maximum of 18 instances and a minimum of 3 instances to run the system.
    
* **Adding a new validator to the system**: We tested the implementation of an automatic validator with Terraform. After the test, we identified some areas for improvement, such as automatic registration of new validators, automatic removal of dishonest validators, and limiting the confirmation time for validator transactions.
    

### **Improvement**

Some areas of improvement we would like to investigate further would be the **response time** of the system. Dealing with distributed, we often face difficulties in network and protocol latencies, which could be an area of research we can go much deeper in.

In addition, another area of improvement would be the transaction verification time. Transactions across distributed systems, especially crosschain interactions, there is a large overhead between block transactions. We can alleviate a good portion of the verification time by immediately transacting across chains right after a block finality.

Likewise, system security would be the next step in improving the system. Currently, the system does not rely on user/password authentication. Instead, it uses digital signatures to authenticate users, so the system does not need to manage the current transaction session of users. The system also relies on consensus on the blockchain for transaction information, so security coverage across the system would be an important detail to look into further.

### Other related topics

The system has many important configuration information that needs to be **secured and managed** by the admin. The system can use Hansicorp Vault or private data in k8s for this purpose. Additionally, for secret keys, the system will import the secret keys of each validator at startup to avoid storing that information in the system.

To ensure data is always **consistent**, we rely on data from the blockchain. Any updates from the system, users, or calculations will be changed if the data from the blockchain changes. Furthermore, data from the blockchain is always the original data for the system to take action.

## Conclusion

Overall, Golang is a great language for building distributed systems due to its simplicity, efficiency, and concurrency support. However, like any language, it has its limitations and challenges. One of the main challenges when working with Golang for distributed systems is session management. Golang does not have built-in session management libraries, and alternative solutions may not always be effective and secure.

Despite these limitations, Golang remains a popular choice for building distributed systems and has many advantages for developers looking to create efficient and scalable systems. As with any technology choice, it is important to carefully consider the trade-offs and limitations before deciding on Golang for a particular project.

### ***Contributing***

*At Dwarves, we encourage our people to read, write, share what we learn with others, and* [***contributing to the Brainery***](https://brain.d.foundation/CONTRIBUTING) *is an important part of our learning culture. For visitors, you are welcome to read them, contribute to them, and suggest additions. We maintain a monthly pool of $1500 to reward contributors who support our journey of lifelong growth in knowledge and network.*

### *Love what we are doing?*

* *Check out our* [***products***](https://superbits.co/)
    
* *Hire us to* [***build your software***](https://d.foundation/)
    
* *Join us,* [***we are also hiring***](https://github.com/dwarvesf/WeAreHiring)
    
* *Visit our* [***Discord Learning Site***](https://discord.gg/dzNBpNTVEZ)
    
* *Visit our* [*GitHub*](https://github.com/dwarvesf)