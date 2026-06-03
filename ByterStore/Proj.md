ByterStore: A Distributed Key–Value Database with Tunable Consistency and Live Update Support

Introduction

ByterStore is a custom-built NoSQL key–value database engineered for modern web applications that demand scalability, responsiveness, and control over consistency guarantees. Unlike traditional key–value systems that focus purely on speed and simplicity, ByterStore introduces advanced mechanisms such as real-time synchronization, adaptive consistency, and extended query support while retaining high performance.

The database is exposed through a web-accessible API, making it suitable as a backend data layer for interactive, multi-user systems.

Design Goals and Unique Features
Live Data Update Notifications

ByterStore supports a subscription-driven update system. When sensitive or high-importance keys are modified, all registered administrators are immediately notified, enabling real-time awareness and coordination across clients.

Configurable Consistency Model

The system allows applications to define their desired consistency behavior at runtime. This enables fine-grained control over the trade-off between consistency, availability, and latency depending on operational needs.

Hybrid Storage and Caching Strategy

To achieve low-latency access, frequently requested keys and repeated queries are cached in memory. Less active data is stored on disk, ensuring efficient use of memory while maintaining scalability under large datasets.

Extended Query Functionality

While preserving the efficiency of a key–value design, ByterStore introduces limited querying over stored values, including:

Attribute-based filtering for JSON-like objects

Range-based queries for numerical fields

This allows more expressive data access without converting the system into a full relational database.

Future Scope: Learning-Based Optimization

The architecture is designed to support future integration of machine learning models that can analyze access patterns and automatically optimize query execution and caching behavior.

Shortcomings in Existing Key–Value Databases
Minimal Query Expressiveness

Most key–value databases are restricted to direct key lookups. Any form of value-based filtering typically requires full data scans, resulting in poor efficiency.

Rigid Consistency Choices

Distributed key–value systems often enforce a fixed balance between consistency and availability, limiting adaptability during network partitions or failures.

Lack of Advanced Operational Features

Common limitations include:

Absence of rollback or recovery mechanisms

No standardized querying interface similar to SQL

Inefficient handling of composite or multi-valued keys

Degraded Performance During Heavy Writes

Systems optimized for read-heavy workloads frequently exhibit bottlenecks when subjected to sustained write operations.

Primitive Data Representation

Most implementations support only flat or primitive data types, offering little native support for structured objects or relationships.

System Breakdown
Core Storage Engine

Technologies Used:

C / C++

STL-based data structures

WebAssembly (WASM) for networking support

Responsibilities:

Data organization and indexing

Memory and disk storage management

Optimization of space and time complexity

Fault detection and error handling

Server-Side Integration Layer

Technologies Used:

Express.js

WebSockets

JSON Web Tokens (JWT)

gRPC

This layer exposes database functionality to clients through REST APIs, manages authentication, and enables real-time communication between the database and frontend applications.

Client-Side Interface

Technologies Used:

React

JavaScript, HTML, CSS

TailwindCSS

The frontend is divided into two independent views:

Administrator Interface

Displays the complete dataset in an organized format

Provides full control over data insertion, modification, and deletion

Receives live notifications for critical updates

User Interface

Represents the consumer-facing application

Currently supports authentication features such as login and registration

Designed for incremental feature expansion