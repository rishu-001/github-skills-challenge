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

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)



