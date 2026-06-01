# ftrace in Linux & Yocto

## Table of Contents

* Introduction
* What is ftrace?
* Why Use ftrace?
* How ftrace Works
* ftrace Architecture
* Where ftrace is Used
* Prerequisites
* Basic Setup
* Common Tracers
* Common Commands
* Understanding ftrace Output
* Common Use Cases
* Advantages
* Disadvantages
* ftrace vs perf
* ftrace vs strace
* Best Practices
* Interview Questions
* Conclusion

---

# Introduction

Debugging and optimizing Linux kernel behavior is a critical task in Embedded Linux and Yocto development.

While tools such as `strace` and `perf` provide application-level and performance-related insights, they cannot always reveal what is happening inside the kernel.

**ftrace (Function Tracer)** is a built-in Linux kernel tracing framework that allows developers to trace kernel functions, scheduler events, interrupts, latency issues, and driver behavior in real time.

It is one of the most powerful debugging tools available for Linux kernel analysis.

---

# What is ftrace?

**ftrace (Function Tracer)** is a kernel-level tracing framework built directly into the Linux kernel.

It is used to:

* Trace kernel function calls
* Analyze scheduling behavior
* Debug drivers
* Measure latency
* Monitor interrupts
* Investigate kernel performance issues

Unlike `strace`, which traces user-space system calls, ftrace traces activities occurring inside the Linux kernel.

---

# Why Use ftrace?

Developers use ftrace to:

* Debug kernel issues
* Analyze boot performance
* Trace driver execution
* Measure scheduling latency
* Detect interrupt delays
* Identify slow kernel functions
* Investigate kernel crashes
* Optimize embedded systems

---

# How ftrace Works

```text
Application
     │
     ▼
System Calls
     │
     ▼
Linux Kernel
     │
     ▼
ftrace Hooks
     │
     ▼
Trace Buffer
     │
     ▼
Trace Output
```

ftrace inserts instrumentation points into kernel functions and records execution information in a trace buffer.

---

# ftrace Architecture

```text
Kernel Function
       │
       ▼
Tracing Hook
       │
       ▼
Ring Buffer
       │
       ▼
Trace File
       │
       ▼
User Reads Results
```

The trace data is stored in a kernel ring buffer and exposed through:

```text
/sys/kernel/debug/tracing/
```

---

# Where ftrace is Used

## Kernel Development

* Function tracing
* Kernel debugging

## Device Driver Development

* Driver initialization debugging
* Interrupt analysis

## Embedded Linux

* Boot time optimization
* Performance analysis

## Yocto BSP Development

* Hardware bring-up
* Driver validation

## Real-Time Systems

* Latency measurements
* Scheduling analysis

---

# Prerequisites

Kernel must be built with:

```config
CONFIG_FTRACE=y
```

Common options:

```config
CONFIG_FUNCTION_TRACER=y
CONFIG_FUNCTION_GRAPH_TRACER=y
CONFIG_DYNAMIC_FTRACE=y
CONFIG_TRACEPOINTS=y
```

Verify:

```bash
cat /sys/kernel/debug/tracing/available_tracers
```

---

# Basic Setup

Mount debug filesystem:

```bash
mount -t debugfs none /sys/kernel/debug
```

Navigate:

```bash
cd /sys/kernel/debug/tracing
```

---

# Common Tracers

Display available tracers:

```bash
cat available_tracers
```

Example:

```text
function
function_graph
irqsoff
wakeup
nop
sched
```

---

## function Tracer

Traces kernel function calls.

Enable:

```bash
echo function > current_tracer
```

---

## function_graph Tracer

Shows function entry and exit.

Enable:

```bash
echo function_graph > current_tracer
```

---

## irqsoff Tracer

Measures periods when interrupts are disabled.

Enable:

```bash
echo irqsoff > current_tracer
```

---

## wakeup Tracer

Analyzes scheduler wakeup latency.

Enable:

```bash
echo wakeup > current_tracer
```

---

# Common Commands

## Start Tracing

```bash
echo 1 > tracing_on
```

---

## Stop Tracing

```bash
echo 0 > tracing_on
```

---

## View Trace Data

```bash
cat trace
```

---

## Clear Trace Buffer

```bash
echo > trace
```

---

## Show Available Events

```bash
cat available_events
```

---

## Enable Scheduler Events

```bash
echo 1 > events/sched/enable
```

---

## Enable IRQ Events

```bash
echo 1 > events/irq/enable
```

---

# Understanding ftrace Output

Example:

```text
bash-1000 [001] .... 123.456:
schedule()
```

Breakdown:

### Process Name

```text
bash
```

Current task.

### CPU

```text
[001]
```

CPU core number.

### Timestamp

```text
123.456
```

Execution time.

### Function

```text
schedule()
```

Kernel function executed.

---

# Common Use Cases

## Analyze Boot Time

Enable function tracing:

```bash
echo function > current_tracer
```

Capture boot activities.

---

## Debug Driver Probe

Filter driver:

```bash
echo my_driver* > set_ftrace_filter
```

View driver functions only.

---

## Measure Interrupt Latency

```bash
echo irqsoff > current_tracer
```

Identify long interrupt-disabled sections.

---

## Trace Scheduler Activity

```bash
echo 1 > events/sched/enable
```

Monitor task switching.

---

## Function Execution Analysis

```bash
echo function_graph > current_tracer
```

Displays:

* Function entry
* Function exit
* Execution duration

---

# Common Trace Files

| File              | Purpose                |
| ----------------- | ---------------------- |
| trace             | Trace output           |
| trace_pipe        | Live trace stream      |
| current_tracer    | Active tracer          |
| available_tracers | Supported tracers      |
| tracing_on        | Enable/disable tracing |
| available_events  | Available events       |
| set_ftrace_filter | Function filtering     |

---

# Advantages

## Built Into Kernel

No external tools required.

---

## Low Overhead

Suitable for embedded systems.

---

## Real-Time Tracing

Can monitor live kernel activity.

---

## Function-Level Visibility

Provides detailed kernel execution information.

---

## Driver Debugging

Excellent for troubleshooting kernel drivers.

---

## Latency Analysis

Useful for real-time systems.

---

# Disadvantages

## Kernel Knowledge Required

Requires understanding of kernel internals.

---

## Large Trace Output

Can generate significant amounts of data.

---

## Configuration Complexity

Many tracers and options available.

---

## Root Access Required

Administrative privileges needed.

---

# ftrace vs perf

| Feature                 | ftrace  | perf    |
| ----------------------- | ------- | ------- |
| Kernel Function Tracing | Yes     | Limited |
| Performance Profiling   | Limited | Yes     |
| Latency Analysis        | Yes     | Limited |
| CPU Hotspots            | No      | Yes     |
| Scheduler Tracing       | Yes     | Limited |

---

# ftrace vs strace

| Feature               | ftrace   | strace  |
| --------------------- | -------- | ------- |
| Kernel Functions      | Yes      | No      |
| System Calls          | Indirect | Yes     |
| User Space Analysis   | No       | Yes     |
| Driver Debugging      | Yes      | Limited |
| Application Debugging | Limited  | Yes     |

---

# Best Practices

* Enable only required tracers
* Clear trace buffer before tracing
* Use filters to reduce output
* Disable tracing after collection
* Use `trace_pipe` for live monitoring
* Combine with perf for performance investigations
* Store trace logs for analysis

Example:

```bash
echo function > current_tracer
echo my_driver* > set_ftrace_filter
echo 1 > tracing_on
```

---

# Interview Questions

### What is ftrace?

A built-in Linux kernel tracing framework used to trace kernel functions, events, interrupts, and scheduling behavior.

---

### Where is ftrace data located?

```text
/sys/kernel/debug/tracing/
```

---

### How do you enable function tracing?

```bash
echo function > current_tracer
```

---

### What is the difference between function and function_graph tracers?

* function → traces function calls.
* function_graph → traces function entry, exit, and execution duration.

---

### How do you start tracing?

```bash
echo 1 > tracing_on
```

---

### What is trace_pipe?

A live streaming interface for viewing trace data in real time.

---

### What is the difference between ftrace and perf?

* ftrace focuses on kernel tracing and latency analysis.
* perf focuses on performance profiling and CPU statistics.

---

# Conclusion

ftrace is a powerful Linux kernel tracing framework used extensively in Yocto and Embedded Linux development. It provides deep visibility into kernel execution, driver behavior, interrupts, scheduling, and latency. Because it is built directly into the Linux kernel, ftrace is one of the most effective tools for debugging and optimizing kernel-level behavior in embedded systems.

### Quick Interview Answer

> **ftrace** is a built-in Linux kernel tracing framework used to trace kernel functions, scheduler events, interrupts, and latency issues. It helps developers debug kernel and driver problems, analyze boot performance, and monitor real-time system behavior. In Yocto and Embedded Linux development, ftrace is commonly used for kernel debugging and performance optimization.
