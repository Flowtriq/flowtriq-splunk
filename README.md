# Flowtriq Splunk Technology Add-on

> **[Integration Guide](https://flowtriq.com/integrations/splunk)** | **[Documentation](https://flowtriq.com/docs)** | **[Sign Up](https://flowtriq.com/signup)**

Splunk TA for receiving and analyzing DDoS detection events from Flowtriq.

## What's Included

- **Field extractions** for the `flowtriq:incident` sourcetype (JSON + regex fallback)
- **Lookup tables** for severity levels and attack family classification
- **Event types and tags** for Splunk CIM compatibility (`network`, `ids`, `attack`)
- **Overview dashboard** with attack timeline, top attack types, severity distribution, targeted nodes, and recent incidents

## Prerequisites

- Splunk Enterprise 8.0+ or Splunk Cloud
- Flowtriq dashboard with the Splunk HEC integration enabled
- A Splunk HEC token configured to accept data

## Installation

### On-Prem Splunk Enterprise

1. Copy the `TA-flowtriq` directory into `$SPLUNK_HOME/etc/apps/`
2. Restart Splunk (`$SPLUNK_HOME/bin/splunk restart`)
3. The app will appear under **Apps** in the Splunk navigation

### Splunk Cloud

1. Package the add-on: `tar -czf TA-flowtriq.tar.gz TA-flowtriq/`
2. Submit the package through Splunk Cloud self-service install or your Splunk Cloud admin
3. The `app.manifest` is included for Splunk Cloud vetting

### Splunk Package (.spl)

```bash
cd flowtriq-splunk
tar -czf TA-flowtriq.spl TA-flowtriq/
```

Then install via **Apps > Install app from file** in the Splunk UI.

## Configuration

### 1. Create a HEC Token in Splunk

1. Go to **Settings > Data Inputs > HTTP Event Collector**
2. Click **New Token**
3. Name it (e.g., "Flowtriq")
4. Set the **Source Type** to `flowtriq:incident`
5. Choose or create an index for Flowtriq events
6. Save and copy the token value

### 2. Configure Flowtriq

1. In the Flowtriq dashboard, go to **Settings > Integrations**
2. Add a **Splunk HEC** integration
3. Enter:
   - **URL**: Your Splunk HEC endpoint (e.g., `https://splunk.example.com:8088/services/collector/event`)
   - **Token**: The HEC token from step 1
   - **Index**: (optional) The index name, or leave blank for the HEC default
   - **Source Type**: Leave as default (`flowtriq:incident`)
4. Test the connection
5. Events will be sent on `attack_start` and `attack_end`

## Event Fields

Each event sent to Splunk contains:

| Field | Type | Description |
|---|---|---|
| `event_type` | string | `attack_start` or `attack_end` |
| `incident_id` | integer | Unique incident identifier |
| `severity` | string | `low`, `medium`, `high`, or `critical` |
| `attack_family` | string | Attack classification (e.g., `udp_flood`, `dns_amplification`) |
| `peak_pps` | integer | Peak packets per second |
| `peak_bps` | integer | Peak bits per second |
| `source_ip_count` | integer | Number of distinct source IPs |
| `confidence` | integer | Detection confidence score |
| `status` | string | Incident status |
| `started_at` | string | ISO 8601 start timestamp |
| `resolved_at` | string | ISO 8601 resolution timestamp (null if ongoing) |
| `node_name` | string | Name of the monitored node |
| `node_ip` | string | IP address of the monitored node |
| `dashboard_url` | string | Direct link to the incident in Flowtriq |

## Dashboard

The **Flowtriq DDoS Overview** dashboard is available under the app and includes:

- Total incident count with color-coded thresholds
- Critical/High incident count
- Peak traffic gauge
- Active node count
- Attack timeline (stacked by severity)
- Top attack types (pie chart)
- Severity distribution (pie chart)
- Top targeted nodes (table with peak traffic stats)
- Recent incidents (table with duration, confidence, and direct links)

All panels support time range and severity filtering.

## Example Searches

```spl
# All incidents in the last 24 hours
sourcetype="flowtriq:incident" event_type="attack_start" earliest=-24h

# Critical incidents by node
sourcetype="flowtriq:incident" severity="critical" | stats count by node_name

# Attack volume over time
sourcetype="flowtriq:incident" event_type="attack_start" | timechart sum(peak_bps) AS total_bps

# Average attack duration
sourcetype="flowtriq:incident" event_type="attack_end"
| eval duration = strptime(resolved_at, "%Y-%m-%dT%H:%M:%S") - strptime(started_at, "%Y-%m-%dT%H:%M:%S")
| stats avg(duration) AS avg_seconds
| eval avg_duration = tostring(avg_seconds, "duration")
```

## Get Started

Start your free 14-day trial at [flowtriq.com/signup](https://flowtriq.com/signup).

## Support

support@flowtriq.com

---

Built by [Flowtriq](https://flowtriq.com) - Real-time DDoS detection and mitigation.
