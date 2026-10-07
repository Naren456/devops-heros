# session-7 : docker-files-images

## Overview
This folder includes Docker image and file-based workflow exercises.

## Objectives
- Validate Docker image build and run steps.
- Test app connectivity and process status.
- Capture screenshots of task outcomes.

## Screenshots

### Fresh verification run (2026-10-07, Cosmic terminal)
![multistage verify](screenshots/00-multistage-verify.png)
Rebuilt `s07-multistage`, ran on host port 8090 (8080 was taken by the
session-06 Java container), `curl` returns
`Hello World from Docker Multi-Stage Build!`, `docker ps` confirms the
running container and port mapping.

### Task 1
![task1 check port](screenshots/task1_check_port.png)
![task1 check process](screenshots/task1_check_process.png)
![task1 image build](screenshots/task1_image_build.png)
![task1 image run](screenshots/task1_image_run.png)

### Task 3
![task3 java app](screenshots/task3_java_app.png)
![task3 node app](screenshots/task3_node_app.png)
![task3 python app](screenshots/task3_python_app.png)
