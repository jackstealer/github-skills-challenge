# AIOps Assessment

## Task 1: Set up and understand the project
I started by looking through the repo and identifying the main parts of the workflow. The important files are:
- `data/service_data.json` - the operational data used for the simulation
- `src/anomaly_detector.py` - checks records for unusual behaviour
- `src/event_producer.py` - publishes anomaly events
- `src/event_topic.py` - holds the in-memory event topic
- `src/event_consumer.py` - reads events from the topic
- `src/aiops_pipeline.py` - runs the end-to-end workflow

The service being monitored is a payment service. The operational problem is that the service can sometimes slow down or fail under load, which shows up as high latency, high resource usage, and error-level logs.

## Task 2: Analyse the operational data
The dataset contains a mix of normal and abnormal observations for the payment service. The fields are:

Metric fields:
- `response_time_ms`
- `cpu_percent`
- `memory_percent`

Log fields:
- `log_level`
- `message`

The `timestamp` field shows when each record was captured. This matters because the data moves through time, and the abnormal behaviour appears in a small window rather than spread across the whole dataset.

The normal records look like successful payment requests. They have lower response times, normal CPU and memory values, and `INFO` log messages. The unusual records have much higher response time and resource usage, and they include `ERROR` entries such as timeouts or database connection issues.

## Task 3: Identify anomalies
I used the provided anomaly detector to process the records. The detector flags records when the metrics exceed the configured thresholds or when there is a relevant error log.

The detected anomalies are:
- `2026-09-20T10:05:00` - payment-service
  - Reasons: high response time, error log detected
  - Relevant values: `response_time_ms = 610`, `log_level = ERROR`, message = `Payment service timeout`
- `2026-09-20T10:06:00` - payment-service
  - Reasons: high response time, high CPU utilization, high memory utilization, error log detected
  - Relevant values: `response_time_ms = 640`, `cpu_percent = 94`, `memory_percent = 91`, `log_level = ERROR`, message = `Database connection timeout`

One limitation is that the detection is rule-based and threshold-driven, so it may not adapt well to changing normal behaviour over time without more advanced baselines or historical comparison.

## Task 4: Verify the event flow
The event-processing flow in this project is:

Operational Data → Anomaly Detection → Event → Producer → Topic → Consumer → AIOps Output

The roles are:
- Producer: creates and publishes the event to the topic
- Topic: stores the event in memory so it can be consumed later
- Consumer: reads the event from the topic
- Event/message: the structured anomaly payload that carries the details about the issue

I checked this flow and confirmed that when an anomaly is detected, it becomes an event, gets published, is consumed from the same topic, and is then available for the downstream AIOps processing step.

## Task 5: Investigate and correct the workflow
I found and fixed a few issues in the workflow:
- the detector was interpreting the log condition incorrectly
- the producer and consumer were not using the same topic in the pipeline
- the project imports needed to work correctly in the normal repo layout


## Task 6: Run the end-to-end pipeline
After the fixes, I ran the full pipeline. The result was successful.

Final output summary:
- 10 records processed
- 2 anomalies detected
- 2 events consumed

This confirms the pipeline is working end-to-end from operational data to final AIOps output.

## Task 7: README and reproduction notes
This repo is a simple AIOps simulation. The goal is to show the basic pattern of detecting a problem, publishing an event, consuming it, and reporting the issue.

To reproduce the workflow:

```bash
cd /workspaces/github-skills-challenge
pytest -q
python -m src.aiops_pipeline
```

These commands validate the project and run the full AIOps pipeline.

## Task 8: Validation
I ran the repository validation after the fixes. The tests pass successfully, and the end-to-end pipeline also runs successfully. This confirms the operational data can be processed, anomaly detection works, anomaly events are generated, the simulated event pipeline works, and the final AIOps output completes as expected.

