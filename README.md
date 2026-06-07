# Exercise 8: Time-Series Architecture & Cloud Ingestion Pipeline

## 🎯 Objective
To provision a MongoDB Atlas cloud cluster, architect a specialized Time-Series collection, and configure Node-RED as a middleware layer to transform raw asynchronous MQTT payloads into strictly formatted time-series documents for persistent cloud storage.

---

## 📖 Theory Brief: The Time-Series Database (TSDB) Paradigm
Standard databases (both SQL and NoSQL) are designed to store discrete entities (like Users or Products). However, IoT sensors generate data differently: they produce continuous, immutable streams of readings over time. If a sensor pings every 1 second, it generates 31 million documents a year. A standard database will experience performance degradation or crash trying to index this volume.

MongoDB solves this challenge using the **Bucket Pattern**. When you designate a collection as "Time-Series," MongoDB stops saving every individual reading as an isolated document on the hard drive. Instead, it groups sequential readings from the same sensor into a single, highly compressed "Bucket" document. To the developer, it looks like normal JSON insertions, but to the server, it is highly compressed, mathematically optimized, and automatically indexed.

---

## 🛠️ Implementation Steps

### Step 1: Architecting the Time-Series Collection in the Cloud
Moving the database architecture to a production-ready cloud environment using MongoDB Atlas:

1. **Configure the Time-Series Collection**:
   * **Database Name**: `smart_campus`
   * **Collection Name**: `environment_telemetry`
   * **Collection Type**: Check the **Time-Series** configuration box.
2. **Define Schema Rules**:
   * **Time Field**: `timestamp` (Tells Atlas which key holds the chronological data axis).
   * **Meta Field**: `sensor_id` (Metadata static label identifying what is generating the data).
   * **Granularity**: Set to `Seconds` (Optimized for frequent ESP32 telemetry pings).

---

### Step 2: Node-RED Middleware Transformation
Raw MQTT payloads received from edge devices (e.g., `{"temperature": 25.5, "humidity": 60}`) are structurally incomplete for a Time-Series Database, which strictly requires a timestamp and explicit metadata. Node-RED functions as the middleware layer to append this missing architecture before payloads reach the cloud.

```mermaid
graph LR
    A[ESP32 / Postman Test] -->|Raw Payload| B(EMQX Broker)
    B -->|campus/zone1/environment| C(Node-RED Function Node)
    C -->|Transformed Schema + Timestamp| D(MongoDB Atlas Cloud)
