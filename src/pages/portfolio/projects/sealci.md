---
layout:  /src/layouts/ProjectLayout.astro
title: 'SealCI'
pubDate: 2025-07-09
description: 'SealCI is a Continuous Integration (CI) system built using Rust and designed with a microservices architecture.'
languages: ["rust", "docker"]
image:
  url: "images/projects/sealci.webp"
  alt: "Thumbnail of SealCI app."  
--- 
# SealCI

SealCI is a Continuous Integration (CI) system built using Rust and designed with a microservices architecture.

## Glossary

- **Action**: A CI atomic unit containing infrastructure, environment, and commands to execute.
- **Action status**: The state of the execution of an action (running, successful, failed).
- **Agent**: A computing node registered with the scheduler.
- **Agent pool**: The set of all registered agents.
- **Pipeline**: A set of actions to be executed, declared as a YAML file.
- **Scheduling**: Selection of an agent to execute an action.
- **Release Agent**: A privileged agent responsible for generating and signing release artifacts upon repository tagging.

For detailed documentation on each component, please refer to the respective markdown files in the `docs/arch` directory.

## Architecture

SealCI is made up of five independant microservices that serve different purpose.
They are pipelined together to create a working CI:

- The Monitor interfaces between the end user, its repository and a Controller.
- The Controller couples to a Scheduler to send actions and receive results and logs.
- The Scheduler registers Agents, sends them actions and transfers results and logs.
- The Agent executes code in the desired environment, and sends back results and logs.
- The Release Agent is a specialized agent that securely generates and signs releases when a Git repository is tagged.

Each service can be hosted, deployed and used separately.

You can find further documentation inside each service's directory, such as `scheduler/README.md` or `scheduler/src/interfaces/README.md`.

### Monitor

The Monitor listens for specific events from remote Git repositories and triggers the controller to launch a CI process based on these events.

- Listening to events from remote Git repositories.
- Exposing a REST API to update the monitoring configuration.
- Recognizing event types and triggering pipelines accordingly.

### Controller

The Controller translates a pipeline declaration file into a list of actions to be executed. It ensures actions are executed in the correct order and provides pipeline state information.

- Users send pipelines containing actions to execute.
- Users can track actions by getting logs and states.
- The controller ensures actions are executed sequentially and handles failures.

The Controller may presently be too tightly coupled with the Scheduler.

### Scheduler

The Scheduler receives a stream of CI actions and tracks a set of CI agents. It selects agents to run the received actions based on their resource capacities and current load.

- Functional without any registered agents.
- Tracks the state and capacity of each registered agent.
- Distributes actions to agents based on resource capacities and load.
- Transfers logs and workload execution result between the agent and controller services.

### Agent

The agent is the powerhouse of SealCI. It receives actions and runs them to complete the operational part of the CI.

- Interfaces with the Docker daemon to execute workloads
- Transfers logs and result back to the controller through the scheduler.

#### Agent lifecycle

- **Registering with a Scheduler**: The agent registers with a scheduler and establishes a bi-directional connection. _Described like this, it's not a loosely-coupled microservice. Which means it may not be following a good philosophy._
- **Health and Death**: The agent streams health and status information to the scheduler.
- **Launching Actions**: The agent creates and runs a container based on the action execution environment configuration, executes commands, and cleans up after completion.

### Release Agent

The **Release Agent** is a specialized agent responsible for securely generating and signing releases when a Git repository is tagged.

- **Triggering**: The release agent is triggered by the Controller when the Monitor detects a new Git tag on the monitored repository.
- **Release Process**:
  - Generates a tarball of the repository at the tagged revision.
  - Signs the tarball with a production-grade signing key.
  - Stores and publishes the signed release artifact securely.
- **Security Guarantees**:
  - Production keys are held exclusively by the Release Agent.
  - Keys never leave the agent’s runtime environment.
  - The agent is run in a dedicated **virtual machine**, ensuring:
    - Protection from container breakout attacks.
    - In-memory key isolation.
    - Hardware-level separation via hypervisor (using Lumper).

🚀 **Developed by Light.**
