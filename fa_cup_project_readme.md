# Real-Time Football Match Telemetry & Spatial Intelligence (FA Cup)

An end-to-end telemetry engineering and spatial analytics platform built natively on **Microsoft Fabric**. This solution ingests high-frequency match event telemetry, streams events into an Eventhouse and KQL database, processes spatial coordinates and player performance data across a Lakehouse Medallion architecture (PySpark), and delivers interactive tactical pitch visuals and threat leaderboards in Power BI via **Direct Lake** mode with zero data refresh lag.

---

## 1. System Architecture & End-to-End Flow

The project follows a modern unified SaaS data pipeline inside Microsoft Fabric:

1. **Telemetry Producer (Stream Source):** A custom event simulator extracts granular match event telemetry from StatsBomb open data (player passes, shots, turnovers, pitch coordinates, and expected goals) and pushes them in live event batches to an Azure Event Hub-compatible custom endpoint.
2. **Fabric Real-Time Intelligence:** An Eventstream (`es_facup_live`) captures the incoming live telemetry and routes records into a Fabric Eventhouse (`live_facup_events`, `kql_facup`) and a dedicated KQL database.
3. **Real-Time KQL Analysis:** A KQL Queryset (`qs_facup_realtime`) performs sub-second tumbling and sliding window aggregations, tracking cumulative expected goals and spatial cluster points as events land.
4. **Lakehouse Medallion Processing:** Ingested events are unified into OneLake and processed across Bronze, Silver, and Gold Delta Lake tables using Apache Spark notebooks in Lakehouse (`lh_facup_analytics`).
5. **Direct Lake Semantic Layer:** A Power BI Semantic Model (`sm_facup_directlake`) links directly to the Gold Delta Parquet files in OneLake without importing data or relying on scheduled refreshes.
6. **Executive Power BI Presentation:** An interactive match report (`FA_Cup`) displays player tactical shapes mapped across pitch coordinates, dynamic player spotlight metrics, and team xG rankings.

---

## 2. Lakehouse Medallion Data Engineering

The pipeline organizes data into three distinct layers to ensure schema validity, data quality, and fast analytical query performance:

### Bronze Layer (`bronze_facup_events`)
* Serves as the raw, append-only landing zone for incoming streaming JSON event feeds.
* Retains complete historical records including raw event timestamps, event identifiers, raw XY location vectors, action types, player tags, and shot outcomes.

### Silver Layer (`silver_facup_events`)
* Cleanses, casts, and normalizes the raw Bronze data.
* Extracts individual horizontal (X) and vertical (Y) coordinates from nested coordinate arrays, validates coordinates against standard pitch dimensions (0 to 120 on X, 0 to 80 on Y), and cleanses missing player strings.
* Enforces typed schemas for match time breakdowns (minutes, seconds) and filters out invalid telemetry noise.

### Gold Layer (`gold_facup_player_impact`)
* A curated analytical dimension and aggregated metrics layer built specifically for the semantic model.
* Calculates the average spatial centroid for every player on the pitch (`avg_pitch_x`, `avg_pitch_y`), providing their true average tactical shape over the course of the match.
* Aggregates key impact metrics per athlete: total match actions, shots attempted, completed passes, and cumulative expected goals (`total_xg`).

---

## 3. Direct Lake Semantic Modeling & DAX Intelligence

Instead of traditional Import mode (which requires dataset refreshes) or DirectQuery (which translates every visual click into slow SQL queries), this solution uses **Direct Lake** mode over OneLake Delta Parquet tables:

* **Performance:** Direct Lake reads parquet column metadata directly into Power BI VertiPaq memory for sub-second visual responsiveness.
* **Dynamic Spotlight DAX Measure:** A context-aware measure detects whether a user has selected an individual athlete on the dashboard. When selected, it formats a dynamic headline string presenting the player's name, team, shot count, and cumulative xG value. If no player is selected, it prompts the user to pick an athlete from the roster.

---

## 4. Power BI Presentation & Analytics Interface

The reporting console features a custom dark tactical aesthetic tailored for sports analytics:

* **Interactive Player Spotlight Banner:** A prominent top headline card displaying real-time metrics for the selected athlete.
* **Average Player Tactical Shape (Spatial Pitch Map):** A scatter plot visual positioned over a pitch outline diagram. Each point represents a player's spatial center-of-gravity based on their average match coordinates (X and Y), visually color-coded by team (Italy vs. Spain).
* **Match xG Leaderboard:** A horizontal bar chart ranking players in descending order of attacking threat based on cumulative match expected goals.
* **Interactive Roster & Team Slicers:** Tile buttons for team selection combined with a scrollable roster search panel, enabling coaches and analysts to isolate individual player profiles instantly.

---

## 5. Fabric Platform Artifacts Reference

* **Eventstream (`es_facup_live`):** Ingestion point for external match telemetry.
* **Eventhouse & KQL Database (`live_facup_events`, `kql_facup`):** Real-time event storage and fast streaming queries.
* **KQL Queryset (`qs_facup_realtime`):** Windowed rolling threat aggregations.
* **Lakehouse (`lh_facup_analytics`):** Storage for Bronze, Silver, and Gold Delta tables.
* **Spark Notebook (`Notebook 1`):** PySpark ETL transformations from raw stream to tactical centroids.
* **Semantic Model (`sm_facup_directlake`):** Direct Lake data model binding Gold tables to Power BI.
* **Power BI Report (`FA_Cup`):** Tactical pitch visualization and match intelligence console.