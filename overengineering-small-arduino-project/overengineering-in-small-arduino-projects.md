---
title: 'Overengineering in Small Projects: Architecture Lessons with an Arduino'
date: '2026-01-29'
description: A story about how a personal Arduino elevator project turned into a practical architecture exercise using events, queues, and a state machine.
lang: en
tags:
    - arduino
    - event-driven
    - fsm
---
## Core Concept

In personal hardware projects, it's common to take the most direct path: open the serial port, send commands, and wait for responses. While this works initially, as the project scales, unpredictable behaviors and maintenance hurdles start to emerge.

This project was born in exactly that context while developing a control system for an Arduino-based elevator. The initial idea was simple: send commands from a server to be executed by the hardware. In practice, however, it turned out to be an exercise in software architecture.

The goal was to build a "bridge" between three distinct parts:

- The Server Responsible for issuing commands.
- The Intermediate Application: Responsible for orchestrating the flow.
- The Arduino Responsible for the physical execution of the elevator.

Instead of direct communication, the proposal was to structure this flow asynchronously using events, queues, and well-defined states, bringing more predictability to the system.

## How It Works

Briefly, the flow follows this path:

Command -> Event -> Queue -> Validation -> Serial -> Execution -> Status

Server publishes a command.
Application receives this event.
Command enters an internal queue.
It is validated and formatted.
It is sent via serial port.
Arduino executes the action.
Status is returned to the server.

In the context of the elevator, this meant transforming actions like *up*, *down*, *open doors*, and *stop* into well-defined events, preventing concurrent or out-of-order commands.

## State Machine

To control this flow, I implemented a Finite State Machine (FSM). Each step of the process was represented by a state, such as:

Idle | Receiving | Processing | Sending | Waiting | Done | Error

In the elevator's case, the FSM helped prevent inconsistent states—like trying to move with the door open, changing direction mid-motion, or accepting commands while an action was still in progress. This became one of the most critical parts of the project.

## Handling Concurrency and Blockage

One of the main challenges was serial communication. If two commands tried to go out at once, the system would crash.

The solution was to create a single processing queue with a dedicated worker for the serial port. This ensured commands were executed sequentially without conflicts. Instead of treating errors as rare exceptions, the project integrated them into the natural flow, making the system much more resilient.

To be honest: today, using an ESP32 with Wi-Fi and MQTT would probably solve half of these issues in much less time. An architecture based on WebSockets or MQTT would eliminate the need for the intermediate app.

However, that would have also eliminated most of the learning. This project served as a laboratory to understand how to organize systems that interact with the physical world. It reinforced something I've noticed over time: it's not the size of the project that defines how much it teaches you, but how much care you put into its structure.

Personal projects, when taken seriously, end up being excellent teachers.

---

*If you've made it this far, you probably also enjoy turning simple ideas into "unnecessarily" complex systems just to learn from them. 😅 If you have suggestions, critiques, or ideas, I'd love to hear them.*