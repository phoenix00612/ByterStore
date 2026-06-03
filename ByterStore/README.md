# ByterStore

**A Scalable Web-Based Key-Value Store with Adaptive Consistency and Real-Time Synchronization**

ByterStore began as a college DBMS project and evolved into a lightweight NoSQL storage engine with a RESTful interface. It is implemented in modern **C++17**, uses a **custom HashMap based on Robin Hood hashing**, and exposes data over HTTP using Crow. ByterStore can be used either as a standalone binary or embedded into other backend services.

**GitHub Repository:** [https://github.com/phoenix00612/ByterStore](https://github.com/phoenix00612/ByterStore)

---

## 🚀 Features

* **Model-based storage**
  Store independent data models (e.g., `users`, `products`) as separate directories under `data/`.

* **Segmented on-disk storage**
  Each model directory contains rolling segment files:

  * `.kv` — append-only data records
  * `.idx` — on-disk index mapping keys to offsets
  * `.bf` — Bloom filter for fast negative lookups

* **Configurable segment sizes** via `config/db.conf`.

* **In-memory caching layer**
  Frequently accessed keys are cached using a Robin Hood hashing–based hashmap.

* **Thread-safe operations**
  Concurrent append, lookup, and delete operations are supported.

* **Pure C++ REST API**
  Exposed using Crow, with no dependency on an external database.

---

## 📦 Tech Stack

* **Core Engine**: C++, STL, `<filesystem>`, `std::thread`, `std::mutex`, custom hashmap library
* **Networking**: Crow (header-only HTTP framework)
* **Build & Tooling**: GNU Makefile, `g++`, `fmt` library
* **Configuration**: JSON (`nlohmann::json`)

---

## 🏁 Quickstart

### 1. Clone & Build

```bash
git clone https://github.com/phoenix00612/ByterStore.git
cd ByterStore
make
```

This builds the ByterStore binary using:

```bash
g++ -std=c++17 -O2 \
    main.cpp config.cpp bloomfilter.cpp segment.cpp segment_mgr.cpp \
    storage_engine.cpp thread_pool.cpp \
    -Iinclude -lfmt -pthread \
    -o byterstore
```

Alternatively, you can download a **prebuilt binary** from the Releases page and extract it.

---

### 2. Configure

Edit **`config/db.conf`** as needed (default shown below):

```json
{
  "data_dir":        "./data",
  "segment_size_mb": 64,
  "file_extension":  ".kv",
  "index_extension": ".idx",
  "bloom_extension": ".bf",
  "bloom_bits_kb":   8,
  "bloom_hashes":    4,
  "thread_pool_size":4
}
```

* `data_dir` stores per-model directories (`users/`, `products/`, etc.)
* Segment sizing and Bloom filter parameters are controlled here

---

### 3. Run

```bash
./byterstore
```

By default, the server listens on **port 8008**.

---

## 📚 API Documentation

All endpoints consume and return JSON.
Base URL: `http://localhost:8008/`

| Method   | Path             | Body (JSON)        | Description                                            |
| -------- | ---------------- | ------------------ | ------------------------------------------------------ |
| `GET`    | `/`              | —                  | List all models (subdirectories).                      |
| `POST`   | `/{model}/{key}` | `{ "key": "..." }` | Create model if missing; insert or update `model/key`. |
| `GET`    | `/{model}`       | —                  | Retrieve all key–value pairs for a model.              |
| `GET`    | `/{model}/{key}` | —                  | Retrieve a single JSON object stored at `model/key`.   |
| `DELETE` | `/{model}`       | —                  | Delete an entire model and its on-disk files.          |
| `DELETE` | `/{model}/{key}` | —                  | Delete a single key from a model.                      |

---

## 🤝 Contributing

1. Fork the repository
2. Create a new feature branch
3. Submit a pull request
