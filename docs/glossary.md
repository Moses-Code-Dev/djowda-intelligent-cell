# Djowda Intelligent Cell Glossary

A curated glossary of terms used across the Djowda Intelligent Cell (DIC) architecture, protocols, testing framework, and data flow cycles.

---

## A

**Android Cell (DIC Instance)**  
A single, autonomous Android-based node in the Djowda ecosystem that participates in the LRS cycle. Each cell acts like a neuron — capable of learning, remembering, and sharing independently.

---

## B

**Broker (MQTT Broker)**  
A message distribution service that routes MQTT messages between DIC cells. Examples include Mosquitto and EMQX.

---

## C

**Conflict Resolution**  
A rule-based process used when two or more cells push differing versions of the same data. May be resolved using timestamps, trust levels, or priorities.

**Cell ID**  
A unique identifier for each DIC instance. It may be used for authentication, message scoping, and trust mapping.

---

## D

**DIC (Djowda Intelligent Cell)**  
The smallest active unit in the Djowda ecosystem. It processes LRS logic and communicates via MQTT and IPFS.

---

## F

**Failsafe Verification**  
A security mechanism that ensures a received message’s origin is valid and the data is schema-compliant.

---

## I

**IPFS (InterPlanetary File System)**  
A decentralized storage protocol used for persistent data sharing in the DIC network, especially for memory exports.

---

## L

**Learn (LRS)**  
The stage where a DIC observes or receives new input data and parses it into local knowledge.

**LRS Cycle**  
The core process of each Intelligent Cell: Learn → Remember → Share. Inspired by biological neurons.

---

## M

**MQTT (Message Queuing Telemetry Transport)**  
A lightweight, publish-subscribe network protocol used by DIC cells for fast, asynchronous message passing.

**Message Queue**  
Temporary storage for outgoing or incoming MQTT messages, used when offline or in flaky networks.

---

## R

**Remember (LRS)**  
The stage where the DIC persists learned data locally using Room DB, and optionally summarizes it for sharing.

**Room DB**  
Android's built-in SQLite abstraction used to store persistent local knowledge in each DIC.

---

## S

**Share (LRS)**  
The stage where a DIC publishes learned and remembered data to external recipients via MQTT or IPFS.

**Snapshot**  
A complete or partial export of a DIC’s internal state, typically used for syncing or historical auditing.

---

## T

**Trust Check**  
The validation of an incoming message’s sender and data against a trusted cell list or cryptographic rules.

---

## U

**UUID (Universally Unique Identifier)**  
Used to uniquely label messages, sessions, or data entities across the DIC ecosystem.

---

## V

**Versioning**  
The practice of tagging data payloads with timestamps or version codes to manage updates, sync, and conflict resolution.

---

## W

**Web3.Storage**  
A platform built on IPFS that allows DICs to upload and retrieve shared state data across the decentralized web.

---

