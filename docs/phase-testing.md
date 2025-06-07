**Phase 1: Single Cell LRS Validation (Unit & Integration Testing on Android)**

**Objective:** Ensure the LRS cycle functions correctly within a single, isolated Android application instance representing one DIC.

**Setup:**
1.  **Develop a Test Android App:** This app will be a simplified version of a DIC (e.g., simulating a "store" app).
2.  **Integrate Core Libraries:**
    *   Room DB for local storage.
    *   MQTT client library (e.g., Paho Android Service).
    *   IPFS/Web3.Storage client library or HTTP client for API interaction.
3.  **Mocking/Test Environment:**
    *   Set up a local MQTT broker (e.g., Mosquitto) or use a public test broker.
    *   Access to a test IPFS node or Web3.Storage account.
    *   Develop mock data generators or ways to manually input data into the app.

**Test Scenarios for the LRS Cycle:**

**(L) Learn - Data Ingestion & Initial Processing:**
1.  **Test Case L1 (MQTT Subscription):**
    *   **Action:** App subscribes to a specific test MQTT topic (simulating a Lobby topic). Publish a well-formed JSON message (as per your defined schema) to this topic from an external MQTT client.
    *   **Expected:** App receives the message, parses it successfully without errors.
    *   **Verify:** Logged data, no crashes, correct parsing of fields.
2.  **Test Case L2 (Local Data Input):**
    *   **Action:** Simulate a local event (e.g., user inputs "new stock arrival" for "tomatoes: 50 units" via a UI element in the test app).
    *   **Expected:** App captures this input and processes it into the internal data model.
    *   **Verify:** Internal data structures reflect the input, data types are correct.
3.  **Test Case L3 (Data Validation):**
    *   **Action:** Send malformed or incomplete data via MQTT or local input.
    *   **Expected:** App handles errors gracefully (e.g., logs error, discards message, doesn't crash).
    *   **Verify:** Error logs, app stability.

**(R) Remember - Local Storage, State Management & Versioning:**
1.  **Test Case R1 (Initial Save):**
    *   **Action:** After a successful "Learn" event (L1 or L2), trigger the "Remember" phase.
    *   **Expected:** Processed data is saved to Room DB with an initial version (e.g., v1.0.0 or timestamp-based) and correct timestamp.
    *   **Verify:** Query Room DB directly to confirm data integrity, version, and timestamp.
2.  **Test Case R2 (Update Existing Data - LWW):**
    *   **Action:** "Learn" a new update for an existing data item (e.g., "tomatoes: stock now 40 units") with a newer timestamp/version.
    *   **Expected:** Room DB updates the existing entry for "tomatoes." The version and timestamp are updated.
    *   **Verify:** Room DB shows new stock, new version, new timestamp. Old data is overwritten.
3.  **Test Case R3 (Stale Data Rejection):**
    *   **Action:** "Learn" an update for "tomatoes" but with an *older* timestamp/version than what's in Room DB.
    *   **Expected:** App rejects the stale update. Room DB remains unchanged for "tomatoes."
    *   **Verify:** Room DB data for tomatoes is not altered. Log indicates stale data rejection.
4.  **Test Case R4 (Snapshot Creation & CID Storage):**
    *   **Action:** Trigger a full state snapshot (e.g., of all inventory). Simulate uploading to Web3.Storage and getting a CID.
    *   **Expected:** The app stores this CID in Room DB, associated with the current overall state version.
    *   **Verify:** Room DB contains the correct CID.

**(S) Share - Data Propagation:**
1.  **Test Case S1 (MQTT Delta Publication):**
    *   **Action:** After data is updated in the "Remember" phase (e.g., R2), trigger "Share."
    *   **Expected:** App publishes a lightweight delta message (correct format, payload, version, timestamp) to a designated MQTT topic.
    *   **Verify:** Use an external MQTT client to subscribe to the topic and validate the published message content.
2.  **Test Case S2 (MQTT Snapshot CID Publication):**
    *   **Action:** After a CID is stored (R4), trigger "Share."
    *   **Expected:** App publishes an MQTT message containing the new CID.
    *   **Verify:** External MQTT client receives the CID message.
3.  **Test Case S3 (IPFS/Web3.Storage Upload):**
    *   **Action:** Trigger a full state snapshot.
    *   **Expected:** App successfully uploads the snapshot data to Web3.Storage and receives a valid CID.
    *   **Verify:** Check Web3.Storage directly (if possible) to confirm file upload. Validate CID format.

**Integrated LRS Flow Test:**
1.  **Test Case LRS1 (End-to-End Single Update):**
    *   **Action:** Input a local data change (Learn) -> Verify it's stored with a new version (Remember) -> Verify the correct delta MQTT message is published (Share).
2.  **Test Case LRS2 (Offline & Sync):**
    *   **Action:** Disable network. Input several local data changes (Learn & Remember). Re-enable network.
    *   **Expected:** App queues "Share" operations while offline. Upon reconnection, it publishes all pending messages.
    *   **Verify:** Monitor MQTT publications after reconnection.

---

**Phase 2: Basic Multi-Cell Interaction (2-3 Android Instances)**

**Objective:** Test data propagation and consistency between a few interacting DICs within a simulated Lobby structure.

**Setup:**
1.  Use 2-3 instances of the Test Android App (on emulators or separate devices).
2.  Configure each to represent a different DIC with a unique `sourceId`.
3.  Ensure they all connect to the same MQTT broker.
4.  Define a simple Lobby structure (e.g., all 3 cells subscribe to the same "Lobby_Test" topics, or Cell A and B are in Lobby 1, Cell C is in Lobby 2, and A & C also subscribe to Lobby 2 due to proximity).

**Test Scenarios:**
1.  **Test Case MC1 (Data Propagation):**
    *   **Action:** Cell A "Learns" new info, "Remembers" it, and "Shares" a delta.
    *   **Expected:** Cell B (if subscribed to the relevant Lobby topic) "Learns" Cell A's delta, "Remembers" Cell A's state (namespaced by `sourceId="CellA"`) with the correct version.
    *   **Verify:** Check Cell B's Room DB for Cell A's data.
2.  **Test Case MC2 (Snapshot CID Propagation & Fetch):**
    *   **Action:** Cell A "Shares" a new snapshot CID.
    *   **Expected:** Cell B "Learns" this CID. Optionally, Cell B attempts to fetch the snapshot from Web3.Storage using the CID and updates its local understanding of Cell A's full state.
    *   **Verify:** Cell B stores CID. If fetching, verify data retrieved matches what Cell A uploaded.
3.  **Test Case MC3 (LWW Across Cells):**
    *   **Action:** Cell A shares version 1 of its state. Cell B receives it. Cell A then shares version 3. Cell B receives it. Cell A then (erroneously or due to network delay) shares version 2.
    *   **Expected:** Cell B correctly processes version 1, then version 3, and correctly discards/ignores the late-arriving version 2 for Cell A's state.
    *   **Verify:** Cell B's Room DB reflects version 3 of Cell A's state.

---

**Phase 3: Stress & Scalability Considerations (Simulated)**

**Objective:** Get an early indication of how the system might behave under more load (this will likely require more than just Android app instances).

**Setup:**
*   Develop lightweight scripts (e.g., Python with Paho-MQTT) to simulate a larger number of DICs publishing and subscribing to Lobby topics.

**Test Scenarios:**
1.  **Test Case SS1 (Lobby Message Throughput):**
    *   Simulate 50-100 cells actively publishing updates to a few shared Lobby topics.
    *   Monitor MQTT broker performance (CPU, memory, message queue length, latency).
    *   Monitor if subscribing clients can keep up.
2.  **Test Case SS2 (Data Convergence Time):**
    *   Update state in one simulated cell.
    *   Measure how long it takes for a significant percentage of other subscribed simulated cells to reflect that change.

---

**Tools & Metrics:**

*   **Tools:** Android Studio (Debugger, Profiler, Logcat), MQTT Explorer (or similar MQTT client), Web3.Storage dashboard/API.
*   **Metrics:**
    *   Successful L/R/S operations vs. failures.
    *   Time taken for each L/R/S step.
    *   Data consistency between cells.
    *   CPU, memory, and battery usage on Android devices.
    *   MQTT message latency.
    *   Error rates and types.

---
