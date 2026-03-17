# ADR-001: Asynchronous Notification Architecture

- **Status:** Accepted
- **Date:** 2026-03-15
- **Author:** Engineering Team
- **Context:** Quiz App API — Email Notification on Quiz Completion

---

## Context

When a user completes a quiz, the system must send an email notification with a results summary. This notification must be handled **asynchronously** and must satisfy the following hard requirements:

1. **Quiz submission must never fail** due to the email service being unavailable.
2. **Notification status must be trackable** (pending, sent, failed).
3. **Failures must be retried automatically** with appropriate backoff.
4. **The solution must fit Hexagonal Architecture** — no infrastructure concerns in the domain layer.
5. The email service is currently **mocked** but must be swappable with a real provider (e.g., SendGrid, SES).

---

## Decision

**Use BullMQ (job queue) backed by Redis for asynchronous notification processing.**

The domain layer raises a `QuizCompletedEvent`. The application layer dispatches it through a `NotificationPort` (interface). An infrastructure adapter (`BullMQNotificationAdapter`) enqueues a persistent job. A dedicated worker processes the job and calls the email service.

---

## Architecture Overview

```
Quiz Submission Request
        │
        ▼
┌─────────────────────────┐
│   SubmitQuizUseCase     │  Application Layer
│   1. Score answers      │
│   2. Persist attempt    │
│   3. Persist notif.     │
│      record (PENDING)   │
│   4. Enqueue job via    │
│      NotificationPort   │
└────────────┬────────────┘
             │ via NotificationPort (interface)
             ▼
┌─────────────────────────┐        ┌──────────────┐
│  BullMQNotificationAdapter│──────▶│    Redis     │
│  (Infrastructure)         │       │  (Job Store) │
└─────────────────────────┘        └──────┬───────┘
                                          │
             ┌────────────────────────────┘
             ▼
┌─────────────────────────┐
│   NotificationWorker    │  Independent consumer
│   1. Read job           │
│   2. Call EmailPort     │
│   3. Update notif.      │
│      status (SENT/FAILED)│
└─────────────────────────┘
```

---

## Evaluated Alternatives

### ❌ Option A: Fire-and-Forget

```ts
// Submit returns 200 before email completes
emailService.send(payload).catch(err => logger.error(err));
```

**Why rejected:**
- **No persistence.** If the process crashes between submission and email, the notification is permanently lost — no retry, no recovery.
- **No status tracking.** Impossible to know if the notification was sent.
- **Uncontrolled concurrency.** Under load, unchecked floating promises can exhaust memory and CPU.
- Directly violates the requirement to track notification status.

---

### ❌ Option B: Webhooks

Webhooks are HTTP callbacks triggered by **external services notifying your API** when something happens (e.g., payment provider → your API). They are not a mechanism for internal async processing.

**Why rejected:**
- Wrong pattern for this problem. Webhooks solve inbound notification from external systems, not outbound async job processing.
- Irrelevant to this use case.

---

### ⚠️ Option C: In-Memory Event Emitter (Domain Events without persistence)

```ts
eventBus.emit('quiz.completed', payload);
eventBus.on('quiz.completed', async (payload) => {
  await emailService.send(payload);
});
```

**Why partially valid but insufficient:**
- ✅ Architecturally clean. Fits Hexagonal perfectly.
- ✅ No external infrastructure dependency.
- ❌ **Not persistent.** Events are lost on server crash or restart.
- ❌ **No retry mechanism.** A failed handler throws and is gone.
- ❌ **No status tracking.** Cannot query whether the notification was actually sent.

**Verdict:** Suitable for intra-application decoupling (e.g., triggering multiple side effects from one event) but **not sufficient alone** for reliable, trackable async processing.

---

### ⚠️ Option D: Full Message Broker (RabbitMQ, AWS SQS, Kafka)

**Why technically valid but over-engineered for this scope:**
- ✅ Durable, persistent, decoupled.
- ✅ Dead Letter Queues for unprocessable messages.
- ✅ Consumer group scaling.
- ❌ **Operational overhead.** Requires provisioning and managing a broker service (RabbitMQ container, SQS IAM config, etc.).
- ❌ **Disproportionate complexity.** The quiz app sends one notification type per attempt — it does not need the guarantees of a distributed message broker.
- ❌ **Kafka** is even more extreme — designed for millions of ordered events per second. Completely disproportionate here.

**Verdict:** The correct answer **at scale** (thousands of notification types, multiple consumer services). Premature for this project's scope.

---

### ✅ Option E: BullMQ + Redis (Selected)

**Why this is the right fit:**

| Requirement | BullMQ + Redis |
|---|---|
| Submission never fails if email is down | ✅ Job is enqueued in Redis atomically before returning `201` |
| Persistent across restarts | ✅ Redis stores jobs durably (AOF persistence) |
| Automatic retries | ✅ Configurable attempts + exponential backoff |
| Status tracking | ✅ Job states: `waiting → active → completed / failed` + own DB column |
| Fits Hexagonal Architecture | ✅ `NotificationPort` is a pure interface in the domain |
| Swappable email provider | ✅ `EmailPort` is injected into the worker |
| Low infrastructure overhead | ✅ Redis is a single, widely-used dependency |
| Visibility & debugging | ✅ Bull Board dashboard for queue monitoring |

---

## Reliability & Fault Tolerance

```ts
const worker = new Worker('notifications', processJob, {
  attempts: 3,
  backoff: {
    type: 'exponential',
    delay: 2000,       // 2s → 4s → 8s
  },
  removeOnComplete: false,   // keep for audit trail
  removeOnFail: false,       // keep for post-mortem debugging
});
```

| Failure Scenario | Behavior |
|---|---|
| Email service temporarily down | Retry 3× with exponential backoff. Auto-recovers when service is back. |
| Email service down for hours | Job remains in `delayed` state in Redis. Retries resume automatically. |
| Worker process crashes mid-job | BullMQ marks job as `stalled` and re-queues it on next worker boot. |
| All retries exhausted | Job moves to `failed` set. Notification record updated to `FAILED`. |
| Redis crashes | Jobs safe if Redis uses AOF persistence. Otherwise, pending jobs may be lost (acceptable risk at this scale). |
| Quiz submission while Redis is down | This is the one edge case to handle explicitly — wrap enqueue in try/catch and still return `201`, with notification recorded as `PENDING` for manual retry. |

---

## Status Tracking

A `notifications` table tracks every notification independently of the job queue:

```
notifications
├── id               UUID, PK
├── attempt_id       FK → quiz_attempts
├── user_id          FK → users
├── status           ENUM: PENDING | SENT | FAILED
├── job_id           BullMQ job ID (for correlation)
├── sent_at          TIMESTAMP, nullable
├── failed_at        TIMESTAMP, nullable
├── failure_reason   TEXT, nullable
└── created_at       TIMESTAMP
```

**Status flow:**

```
[Quiz Submitted] → PENDING (record created + job enqueued)
                      │
          ┌───────────┴───────────┐
       Worker succeeds        All retries exhausted
          │                       │
        SENT                    FAILED
    (sent_at set)         (failed_at + reason set)
```

---

## Hexagonal Architecture Fit

The domain and application layers remain free of infrastructure details:

```ts
// core/ports/NotificationPort.ts  (domain — pure interface)
export interface NotificationPort {
  enqueueQuizCompletedNotification(data: QuizCompletedNotificationData): Promise<void>;
}

// core/ports/EmailPort.ts  (domain — pure interface)
export interface EmailPort {
  sendQuizResults(payload: QuizResultsEmailPayload): Promise<void>;
}

// infrastructure/queue/BullMQNotificationAdapter.ts  (infrastructure — implements port)
export class BullMQNotificationAdapter implements NotificationPort {
  constructor(private readonly queue: Queue) {}

  async enqueueQuizCompletedNotification(data: QuizCompletedNotificationData): Promise<void> {
    await this.queue.add('quiz-completed', data);
  }
}

// infrastructure/email/MockEmailAdapter.ts  (infrastructure — swappable)
export class MockEmailAdapter implements EmailPort {
  async sendQuizResults(payload: QuizResultsEmailPayload): Promise<void> {
    // Simulate sending — swap with SendGrid/SES adapter in production
    console.log(`[MOCK EMAIL] Sending results to ${payload.userEmail}`);
  }
}
```

---

## Consequences

**Positive:**
- Quiz submission is fully decoupled from email delivery. `201 Created` is returned before any email processing begins.
- Failures in the notification layer do not affect the user experience.
- Full observability: every notification has a trackable status with timestamps and failure reasons.
- The email provider can be swapped (mock → real) by changing only the `EmailPort` adapter, with zero changes to domain or application code.

**Negative / Trade-offs:**
- Redis is now a required infrastructure dependency. It should be containerized alongside the app (e.g., `docker-compose`).
- Eventual consistency: the notification is guaranteed to be delivered, but not instantly. Acceptable given the use case (email results summary — not time-critical).
- If Redis is unavailable at enqueue time, the job cannot be queued. Mitigation: wrap enqueue in a try/catch boundary and surface the `PENDING` status for later admin retry.

---

## References

- [BullMQ Documentation](https://docs.bullmq.io/)
- [Redis AOF Persistence](https://redis.io/docs/management/persistence/)
- [Bull Board — Queue Dashboard](https://github.com/felixmosh/bull-board)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [Hexagonal Architecture (Ports & Adapters)](https://alistair.cockburn.us/hexagonal-architecture/)
