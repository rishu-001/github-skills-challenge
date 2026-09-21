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

---

&copy; 2025 GitHub &bull; [Code of Conduct](https://www.contributor-covenant.org/version/2/1/code_of_conduct/code_of_conduct.md) &bull; [MIT License](https://gh.io/mit)



