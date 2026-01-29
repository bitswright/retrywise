# `retrywise`

Modern distributed systems rely heavily on external APIs and internal services that are unreliable by nature. Naive retry mechanisms often make things worse by causing retry storms, violating rate limits, and increasing tail latency.

`retrywise` is a Go library that provides intelligent, adaptive retry and rate‑limit handling for HTTP calls. Instead of blindly retrying, `retrywise` makes runtime decisions based on error types, rate‑limit signals, latency budgets, and recent failure patterns.

## Problem Statement
Most retry implementations suffer from one or more of the following issues:
- Fixed retry counts instead of time‑based budgets
- No awareness of API rate limits
- Retry storms during partial outages
- Lack of circuit breaking
- Poor observability into retry behavior
These shortcomings often lead to cascading failures and degraded system reliability.

## Solution
`retrywise` acts as a smart wrapper around Go's `net/http` client. It introduces:
- Time‑based retry budgets instead of fixed retry counts
- Exponential backoff with jitter to avoid synchronized retries
- Rate‑limit–aware retry decisions (e.g. Retry-After, remaining quota)
- Circuit breaker to fail fast during downstream outages
- Context‑aware cancellation for correctness under load
The library is designed to be embedded directly into services with zero additional network hops.

## Key Features

1. Adaptive Retry Engine
Retries are attempted only when they are likely to succeed, based on:
- Error classification (timeouts, 5xx, 429, etc.)
- Remaining retry budget
- Circuit breaker state

2. Time‑Based Retry Budget
Instead of retrying a fixed number of times, `retrywise` retries until a maximum elapsed duration is reached. This provides predictable upper bounds on request latency.

3. Rate‑Limit Awareness
`retrywise` inspects common rate‑limit headers such as:
- `Retry-After`
- `X-RateLimit-Remaining`
When rate limits are near exhaustion, retries are delayed or skipped entirely.

4. Circuit Breaker
`retrywise` implements a simple circuit breaker state machine:
- `Closed` – normal operation
- `Open` – requests fail fast
- `Half‑Open` – limited test requests allowed
This prevents cascading failures during partial outages.

5. Observability
The library exposes structured logs and hooks for collecting metrics, including:
- Retry count
- Success after retry
- Circuit breaker state transitions
- Rate‑limited requests


## Failure Scenarios Handled
- Network timeouts
- Transient 5xx errors
- API rate limiting (429)
- Partial outages
- Slow downstream dependencies


## Design Decisions & Trade‑offs
### Why Retry Budget Over Retry Count?
Retry counts ignore latency variability. A time‑based retry budget ensures predictable request behavior even under slow downstream conditions.

## What I Would Improve at Scale
- Distributed circuit breaker using *Redis* or *etcd*
- Per‑endpoint adaptive retry policies
- Dynamic configuration reload
- Prometheus / OpenTelemetry integration
- Token‑bucket–based rate limiting
