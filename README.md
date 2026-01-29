# NodeJs_Architecture-# 🚀 Node.js Execution Model (Event Loop, libuv & Heavy Tasks)

This document explains **how Node.js executes code**, how **libuv works**,  
why **CPU-heavy JavaScript blocks the event loop**, and how **I/O is handled efficiently**.

---

## 1️⃣ Node.js Basics

- Node.js is **single-threaded at the JavaScript level**
- The **main thread**:
  - Executes JavaScript
  - Runs the **event loop**
- Heavy **I/O operations** are delegated to **libuv**
- **CPU-heavy JavaScript** blocks everything

---

## 2️⃣ High-Level Architecture

```mermaid
flowchart TD
    JS[JavaScript Code] --> MT[Main Thread]
    MT --> EL[Event Loop]

    EL -->|Async I/O| LUV[libuv]
    LUV --> TP[Thread Pool]
    LUV --> OS[OS Async APIs]

    TP --> Q[Callback Queue]
    OS --> Q
    Q --> EL


3️⃣ Single-Threaded JavaScript Execution
