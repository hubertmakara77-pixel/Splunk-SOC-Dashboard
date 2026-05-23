# Splunk SOC Analytics Dashboard - BOTS v1 Threat Hunting

## 📝 Project Overview

This project demonstrates the creation of an interactive Security Operations Center (SOC) Dashboard inside **Splunk Enterprise** using the famous **Boss of the SOC (BOTS) v1** dataset provided by Splunk. The goal of this project was to ingest raw, multi-source security logs, normalize the data, filter out background noise, and build an interactive visualization for security analysts to detect and track global network threats.

---

## 🛠️ Splunk Power User Skills Demonstrated

This project was built to align with the **Splunk Core Power User** competencies, showcasing advanced search optimization and knowledge object management:

* **Advanced SPL (Search Processing Language):** Extensive use of data transformation commands (`stats`, `timechart`, `sort`, `head`).
* **Geospatial Analytics:** Mapping attacker locations using `iplocation` and `geostats`.
* **Dashboard Interactivity (Tokens):** Implementing a centralized `Shared Time Picker` utilizing tokens to drive real-time data filtering across all panels simultaneously.
* **Knowledge Objects & Optimization (Macros):** Creating modular search macros (`ignore_known_scanners`) to dynamically filter out False Positives and corporate noise.
* **Troubleshooting & Administration:** Manual app deployment via the backend file structure and handling compressed archive formats (`.tar.gz` / `.tar`).

---

## 📊 Dashboard Architecture & Panels

### 1. Global Traffic Source Map (Suricata IDS)

**SPL Query:**
```splunk
index=botsv1 sourcetype=suricata | iplocation src_ip | geostats count
```

**Description:** Leverages Splunk's internal MaxMind database to resolve raw IP addresses into geographical coordinates, plotting active network connections on a global cluster map to identify anomaly traffic origin.

### 2. Top 5 Network Aggressors (Noise Filtered)

**SPL Query:**
```splunk
index=botsv1 sourcetype=suricata `ignore_known_scanners` | stats count by src_ip | sort - count | head 5
```

**Description:** Displays the most aggressive source IPs hitting the network. It demonstrates defensive analyst logic by utilizing a custom macro to exclude benign high-volume traffic (such as Google DNS `8.8.8.8`).

### 3. Attack Timeline Analysis (Timechart)

**SPL Query:**
```splunk
index=botsv1 sourcetype=suricata `ignore_known_scanners` | timechart span=1h count by src_ip limit=5 useother=f
```

**Description:** A visualization showing attack spikes over time. Optimized with Power User arguments (span=1h, limit=5, useother=f) to keep the timechart clean and actionable for incident response.

<img width="1618" height="248" alt="Zrzut ekranu 2026-05-23 210904" src="https://github.com/user-attachments/assets/ae531997-b1ac-449c-a7a8-5ea2a028751b" />
<img width="1618" height="482" alt="Zrzut ekranu 2026-05-23 210909" src="https://github.com/user-attachments/assets/4d218d66-f25b-4f53-af3b-aa98f782697d" />
<img width="1618" height="314" alt="Zrzut ekranu 2026-05-23 210917" src="https://github.com/user-attachments/assets/9be6236b-3dce-48e5-a1b2-5dddba8a85dc" />



---

## 💡 Key Analytical Insights & Challenges

### The Problem
During the initial deployment, the "Top Aggressors" panel was dominated by public DNS traffic (specifically Google's 8.8.8.8 with nearly 400k events).

**The Challenge:** High-volume legitimate traffic was blinding the dashboard from seeing actual brute-force and scanning attempts.

### The Solution
I engineered a Splunk Macro named `ignore_known_scanners` defined as:
```splunk
NOT src_ip IN ("127.0.0.1", "8.8.8.8")
```

This successfully suppressed the noise, immediately revealing suspicious external and internal pivoting IPs at the top of the analyst's queue.

---

## 🚀 How to Replicate This Project

1. **Install Splunk Enterprise** locally.

2. **Download and extract the BOTS v1 Dataset** into your `$SPLUNK_HOME/etc/apps/` directory.

3. **Restart Splunk** and ensure the data is indexed under `index=botsv1`.

4. **Create a new Classic Dashboard** in Splunk, go to the Source tab, and paste the contents of the `soc_dashboard_botsv1.xml` file provided in this repository.

---

## 📁 Project Structure

```
.
├── README.md
├── soc_dashboard_botsv1.xml      # Dashboard configuration
└── macros/
    └── ignore_known_scanners.spl  # Custom macro definition
```

---

## 📚 References

* [Splunk Boss of the SOC (BOTS) v1](https://www.splunk.com/en_us/products/bots.html)
* [Splunk Search Processing Language (SPL) Documentation](https://docs.splunk.com/Documentation/Splunk/latest/SearchReference/CommandsbyCategory)
* [Splunk Dashboard Visualization Guide](https://docs.splunk.com/Documentation/Splunk/latest/Viz/Visualizations)
