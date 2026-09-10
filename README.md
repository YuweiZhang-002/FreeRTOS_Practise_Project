# FreeRTOS_Practise_Project
This project aims to understand FreeRTOS mechanism using STM32F407 MCU. Reference only

## RTOS Architecture Guidance (Design-first)
Because this project focuses on understanding RTOS mechanisms, prefer **design decisions** before coding:

### 1) Priority allocation strategy
- Start from deadlines and safety impact, not from “importance by feeling”.
- Keep only a small number of distinct priority levels (for example: critical control > communication > logging/background).
- Use **event-driven tasks** (queues/semaphores/notifications) instead of fast polling loops, so high-priority tasks are responsive without starving others.
- Avoid putting long or blocking operations in the highest-priority tasks.
- Revisit priorities after measurement (`vTaskGetRunTimeStats`, trace tools, stack high-water marks).

### 2) Mutex and shared-resource strategy
- Use a **mutex** for exclusive ownership of shared peripherals/data (I2C, SPI, UART print buffer, shared state).
- Keep mutex hold time short: lock late, unlock early.
- Do not call long-delay/blocking APIs while holding a mutex.
- Always define and document lock ordering when multiple mutexes exist (prevents deadlock).
- Prefer message passing to a dedicated owner task when contention is high.
- Never use a mutex in ISR context; use ISR-safe primitives (`...FromISR`) and defer work to tasks.

### 3) Recommended design workflow
1. Draw task/resource map: task period/trigger, worst-case execution time, shared resources.
2. Assign initial priorities and lock ownership rules.
3. Validate under stress (burst traffic + worst timing).
4. Adjust priorities/critical sections based on measured latency and CPU usage.

### 4) Typical anti-patterns to avoid
- Too many adjacent priority levels with unclear rationale.
- Global “debug print” mutex held for long formatted output.
- Mixed ownership of the same peripheral from multiple tasks without one clear policy.
- Treating mutexes as signaling tools (use binary semaphores/notifications instead).
