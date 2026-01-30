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
## The Core Idea

In personal hardware projects, it’s very common to start with the most straightforward approach: open the serial port, send commands, and wait for responses. At first, it works. But as the project grows, issues begin to appear: blocking, message loss, unpredictable behavior, and maintenance difficulties.

This project emerged exactly in that context, during the development of an elevator control system using Arduino. The initial idea was simple: send commands from a server and execute them on the hardware. In practice, it turned into a small architecture exercise.

The goal was to create a “bridge” between three different parts:

- A server, responsible for emitting commands  
- An intermediary application, responsible for organizing the flow  
- An Arduino, responsible for the physical execution of the elevator  

Instead of having everything communicate directly, the proposal was to structure the flow asynchronously, using events, queues, and well-defined states, bringing more predictability and reducing common problems in projects like this.

## How the System Works

In short, the flow follows this path:

Server publishes a command.  
Application receives the event.  
Command enters an internal queue.  
It is validated and formatted.  
It is sent through the serial port.  
Arduino executes the action.  
Status returns to the server.

In the context of the elevator, this meant transforming actions like going up, going down, opening doors, and stopping into well-defined events, avoiding concurrent or out-of-order commands.

This decoupling was essential to keep the system stable. Each part had a clear responsibility, and the use of events allowed them to operate independently.

## The Importance of the State Machine

To control this flow, I implemented a finite state machine (FSM).

Each stage of the process was represented by a state, such as:

- Idle  
- Receiving command  
- Processing  
- Sending  
- Waiting for response  
- Finished  
- Error  

In the case of the elevator, the FSM helped prevent situations like trying to move up with the door open, changing direction mid-movement, or accepting commands while still executing.

This prevented the system from entering inconsistent states. The FSM ended up being one of the most important parts of the project.

## Dealing with Concurrency and Blocking

One of the main problems was serial communication.

If two commands tried to be sent at the same time, the system would break. In the elevator, this could mean unexpected movements or loss of control. The solution was to create a single processing queue with a dedicated serial worker. As a result, commands were executed in order, without conflicts.

Instead of treating errors as rare exceptions, the project began to see them as a natural part of the flow, which made the system much more predictable.

The most interesting part of this project was not the final result, but the process. It showed that small projects sometimes also need architecture, and that concepts seen in larger systems naturally emerge, even in personal projects.

## Limitations and Simpler Paths

To be honest: today, using an ESP32 with Wi-Fi and MQTT would probably solve half of this in much less time.

An architecture based on WebSocket or MQTT would eliminate the need for the intermediary application, but it would also remove much of the learning experience. This project was, above all, a laboratory to understand how to organize systems that interact with the physical world.

More than controlling an elevator with Arduino, this project served to experiment, make mistakes, adjust, and better understand how event-driven systems work in practice.

It reinforced something I’ve been realizing over time: it’s not the size of a project that defines how much it teaches, but how much you care about structuring it well.

Personal projects, when taken seriously, end up being excellent teachers.

---

*If you’ve made it this far, you probably also enjoy turning simple ideas into unnecessarily complex systems and learning from them. 😅*
