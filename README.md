# GitHub Challenge

<img src="https://octodex.github.com/images/Professortocat_v2.png" align="right" height="200px" />

Hey there!

Your challenge is ready.
Follow the instructions provided for this challenge and complete the required tasks in this repository.

Make sure your work is committed and pushed to your repository before submission.

Good luck!

## Service Overview
This project monitors a payment service called `payment-service`. It looks at system data such as response time, CPU use, memory use, and log events to understand whether the service is healthy or degrading.

## Repo Map
- Operational data: `data/service_data.json` contains the service records used for the simulation.
- Metrics and logs: the records include response time, CPU, memory, and log level data from the service.
- Anomaly detection: `src/anomaly_detector.py` checks the data for unusual patterns and marks issues as anomalies.
- Event production: `src/event_producer.py` sends the detected anomalies into a topic.
- Event topics: `src/event_topic.py` stores the in-memory events for the pipeline.
- Event consumption: `src/event_consumer.py` reads the events after they are produced.
- Final AIOps processing: `src/aiops_pipeline.py` brings everything together to load data, detect problems, and report the results.

## Operational Problem Being Addressed
The service is having performance and reliability issues. The data shows slow response times, high CPU and memory usage, and error logs, which indicate that the service may be failing or about to fail.

## Purpose of AIOps in This Assessment
AIOps helps turn raw system data into useful alerts. Instead of manually reading logs and metrics, the project uses automated detection to identify issues faster and support better operational response.



Task 2:

## Analysis of Logs and Metrics in the Sample Data
The repo contains a small synthetic dataset in `data/service_data.json` that represents the behavior of a service over time.

### Metric fields
These fields are operational metrics:
- `response_time_ms`: how long requests take to complete
- `cpu_percent`: CPU usage of the service
- `memory_percent`: memory usage of the service

These values are numeric and show how the service is performing under load.

### Log-related fields
These fields contain log information:
- `log_level`: the severity of the event, such as `INFO` or `ERROR`
- `message`: the message text associated with the event

Together, these fields describe what happened and how serious it was.

### How timestamps are used
- `timestamp` shows when each record was generated.
- The values are ordered in time and are used to track service behavior over a short period.
- In this dataset, the timestamps move from `10:00:00` to `10:09:00`, so the records form a simple time series.

### Normal behavior
The early records look normal:
- response times stay around 120-145 ms
- CPU stays around 42-50%
- memory stays around 51-57%
- log level is mostly `INFO`
- messages say the payment request was processed successfully

This indicates the service is operating normally and handling requests without major issues.

### Unusual behavior
The later records clearly show abnormal service behavior:
- at `10:05:00`, response time jumps to 610 ms
- CPU rises to 75% and memory to 70%
- the log level changes to `ERROR`
- the message reports a timeout

The next record at `10:06:00` is even more severe:
- response time reaches 640 ms
- CPU reaches 94%
- memory reaches 91%
- error logs continue

These values suggest a service degradation or failure state, where the application is becoming slow and overloaded.

### Summary
The dataset shows a service that is healthy at first, then becomes unstable as latency, CPU, memory, and error logs all worsen together. This is the kind of pattern AIOps is designed to detect and alert on.







Task 3:

## Anomaly Detection Review
Using the provided detector in `src/anomaly_detector.py`, the actual analysis of the sample data flags two abnormal records.

### Anomalies detected
- `2026-09-20T10:05:00`: flagged as `ANOMALY`
  - Relevant metrics: `response_time_ms = 610`, `cpu_percent = 75`, `memory_percent = 70`
  - Log info: `log_level = ERROR`, message = `Payment service timeout`
  - Reason: `High response time`

- `2026-09-20T10:06:00`: flagged as `ANOMALY`
  - Relevant metrics: `response_time_ms = 640`, `cpu_percent = 94`, `memory_percent = 91`
  - Log info: `log_level = ERROR`, message = `Database connection timeout`
  - Reasons: `High response time`, `High CPU utilization`, `High memory utilization`

### Expected anomaly missed?
No clear expected anomaly was missed in this sample dataset. The abnormal records are obvious and are detected by the component as soon as the response time and resource thresholds are crossed.

### Normal event incorrectly flagged?
No normal event appears to have been incorrectly flagged. The records before `10:05:00` stay within stable ranges and do not trigger the rule set.

### One limitation / improvement
A simple threshold-based detector can miss gradual problems that do not cross a fixed threshold immediately. A better approach would be to add trend-based detection, such as comparing current values against recent service history, so the system can detect early warning signals before the metric spikes become severe.











Task 4:

## Event Flow Verification
To verify the event-processing path, I ran the actual workflow using the project code and the provided dataset.

### Execution result
Command used:
`PYTHONPATH=src python3 - <<'PY' ... run_pipeline('data/service_data.json') ... PY`

Observed result:
- `records_processed`: 10
- `anomalies_detected`: 2
- `events_consumed`: 0

The runtime output showed:
- anomaly records were created for `2026-09-20T10:05:00` and `2026-09-20T10:06:00`
- the detector successfully produced anomaly event objects
- the final consumer returned an empty list

This means the detection step worked, but the full event pipeline did not complete end-to-end.

### Verified event-flow steps
1. Detection step
   - `src/anomaly_detector.py` inspects each record in `data/service_data.json`.
   - If a record exceeds thresholds or contains an error log, it returns an anomaly event.

2. Event message creation
   - The anomaly event is a dictionary containing at least:
     - `timestamp`
     - `service`
     - `type`
     - `reasons`
     - `source`
   - This object is the message that moves through the pipeline.

3. Producer role
   - `src/event_producer.py` is the component that takes the anomaly message and publishes it to a topic.
   - In the workflow, the producer is created with `EventProducer(producer_topic)` where `producer_topic = EventTopic("service-events")`.

4. Topic role
   - `src/event_topic.py` holds the in-memory event stream.
   - It stores messages in `self.messages` and exposes `publish()` and `get_messages()`.
   - The topic acts as the message bus between producer and consumer.

5. Consumer role
   - `src/event_consumer.py` reads from a topic using `consume()`.
   - In the workflow, the consumer is created with `EventConsumer(consumer_topic)` where `consumer_topic = EventTopic("anomaly-events")`.

### Why the flow failed in this repo
The detected anomaly events were published to the `service-events` topic, but the consumer was reading from a different topic named `anomaly-events`.

This creates a split in the event flow:
- producer writes to one in-memory topic
- consumer reads from another in-memory topic
- no message reaches the consumer, so `events_consumed` stays empty

The issue is visible in `src/aiops_pipeline.py`:
- `producer_topic = EventTopic("service-events")`
- `consumer_topic = EventTopic("anomaly-events")`

Because those are separate objects, the same event never travels through the full processing path.

### Result of the verification
The repository currently demonstrates a partial event pipeline:
- anomaly detection works
- producer logic works
- topic storage works
- consumer logic works only when connected to the same topic

The complete end-to-end flow is not currently working because the producer and consumer are not attached to the same event topic.

### Role summary
- Producer: publishes each detected anomaly as a message
- Topic: stores messages in memory for downstream processing
- Consumer: reads messages from the topic
- Message: the anomaly event payload carrying service details and reasons for the alert

This verification confirms the core AIOps simulation design, while also showing a real integration issue in the current implementation.










Task 5:

## Workflow Investigation and Correction
I investigated the event flow to confirm where the processing chain was breaking and applied the smallest correct fix that fits the existing architecture.

### Problem identified
The workflow was not fully connected. The producer and consumer were using different in-memory topic objects, so the anomaly message was published to one topic and never read from the other.

### Affected component
The issue was in `src/aiops_pipeline.py`, which created separate topics for the producer and consumer instead of reusing a single shared topic.

### Cause
The pipeline created:
- `producer_topic = EventTopic("service-events")`
- `consumer_topic = EventTopic("anomaly-events")`

Because these were different objects, there was no shared message path between them.

### Correction applied
I corrected the workflow so the producer and consumer share the same `EventTopic` instance. This keeps the design simple and works with the current event-stream simulation architecture.

### Verification
I re-ran the workflow after the fix.

Observed result:
- `records_processed`: 10
- `anomalies_detected`: 2
- `events_consumed`: 2

This confirms that the anomaly messages now travel from detection to publication to consumption successfully.

### Final outcome
The complete AIOps event flow is now working under the existing design:
- anomaly detection identifies issues
- the producer publishes the event
- the topic stores the message
- the consumer reads the message back

The producer, topic, consumer, and message roles are all now working as intended in the current architecture.














DO TSAK 8
Task 6:

## Execute the End-to-End Pipeline
After investigating and correcting the workflow, I executed the complete AIOps pipeline to verify the full operational path from data ingestion to final output.

### End-to-end flow demonstrated
Operational Data → Anomaly Detection → Event → Producer → Topic → Consumer → AIOps Output

### Verified execution result
I ran the workflow using the repository’s sample data in `data/service_data.json`.

Observed outcome:
- `records_processed`: 10
- `anomalies_detected`: 2
- `events_consumed`: 2

The final output shows two anomalies were detected and successfully consumed:
- `2026-09-20T10:05:00` — `Payment service timeout`
- `2026-09-20T10:06:00` — `Database connection timeout`

### What was verified
1. Operational data is processed.
2. Anomalous behaviour is detected.
3. An anomaly event is generated.
4. The event is published by the producer.
5. The event is stored on the topic.
6. The event is consumed by the consumer.
7. The final output represents the detected operational issue.

### Role of each component in the workflow
- Producer: `src/event_producer.py` publishes the anomaly event to the topic.
- Topic: `src/event_topic.py` stores the in-memory event stream.
- Consumer: `src/event_consumer.py` reads the message back from the topic.
- Message: the anomaly object carrying the timestamp, service name, type, reasons, and source telemetry.

### Final status
The end-to-end pipeline now works successfully under the current architecture and confirms the complete AIOps event-processing flow.

Task 7:

## Reproduction Steps for Another User
To reproduce this demonstration, another user can follow the same workflow used in this project.

### Steps to reproduce
1. Open the repository in the project environment.
2. Review the operational data in `data/service_data.json`.
3. Run the anomaly detection logic against the sample records.
4. Confirm that abnormal records are identified using the thresholds in `src/anomaly_detector.py`.
5. Run the pipeline in `src/aiops_pipeline.py`.
6. Confirm that the detected anomalies are published by the producer to the shared topic.
7. Confirm that the consumer reads the same messages back from that topic.
8. Review the final output to confirm that the detected issue matches the operational problem shown in the data.

### What another user should expect
A successful run should show:
- `records_processed`: 10
- `anomalies_detected`: 2
- `events_consumed`: 2

This indicates the end-to-end pipeline is functioning correctly and that the detected anomalies have successfully traversed the event-processing workflow.

### Additional note
The key requirement is that the producer and consumer use the same topic instance. This is the critical design detail that makes the message flow work in the current in-memory event simulation.

Task 8:

## Validate and Save Your Work
Use the project checks and validation steps to confirm that the implementation behaves as expected before submitting the work.

### Validation checklist
Confirm that:
- the operational data can be processed
- anomaly detection behaves as expected
- anomaly events are generated
- events can move through the simulated event pipeline
- consumers can process the generated events
- the final AIOps workflow completes successfully

### Validation result
The project was validated by running the workflow against the provided dataset and confirming that:
- `records_processed`: 10
- `anomalies_detected`: 2
- `events_consumed`: 2

This confirms the pipeline completes successfully and the detected operational issue is reported in the final output.

### Final review before submission
1. Review all modified files.
2. Confirm that only the required assessment edits are included.
3. Confirm that the README documents the project scenario, analysis, findings, workflow correction, execution result, and reproduction steps.
4. Save the final version of the project.
5. Commit the changes with a clear commit message.
6. Push the branch to GitHub.
7. Confirm the repository reflects the final validated state.

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)



