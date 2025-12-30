# FactStore - Official Documentation

A lightweight, Kotlin-first event store designed for clarity, correctness, and extensibility — built on a clean specification and powered by FoundationDB. 🚀

## What is FactStore?

The _FactStore_ project is a collection of libraries to build flexible event-sourced applications, including a specification for event sourcing and event streaming, as well as a backing implementation of that specification powered by FoundationDB. 

### Subproject: `fact-store-specification`

This subproject defines the core contracts and behavioral rules for a FactStore implementation. It is written in Kotlin and establishes the APIs for:

* Appending events (also called facts)
* Reading streams of facts
* Subscribing to live or historical streams
* Managing event ordering and consistency guarantees

This module is intentionally implementation-agnostic, serving as the foundation for any backend.

Other subprojects include: 

* _FactExplorer_: A UI tool to explore and manage fact stores. 

### Subproject: `fact-store-foundationdb`

An implementation of the specification using FoundationDB as the storage engine.
This module provides:

* A stateless event-sourcing layer built on FoundationDB’s transactional model 
* Strong consistency and ordered event writes
* Efficient range reads and streams
* A clean separation between domain logic and storage mechanics

If you want a production-ready FactStore backed by FoundationDB, this is the module for you.
