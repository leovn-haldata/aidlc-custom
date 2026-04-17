# Event Contracts: <change-id>

## Change Reference
- **Design**: [design.md](../design.md)
- **Mode**: full-spec

## Events

### Event: _event.name_

| Attribute | Value |
|-----------|-------|
| **Name** | _domain.entity.action_ (e.g., `user.account.created`) |
| **Producer** | _Component that emits this event_ |
| **Consumers** | _Components that listen for this event_ |
| **Trigger** | _When is this event emitted?_ |
| **Idempotent** | Yes / No |

**Payload Schema**:

```json
{
  "event_id": "uuid",
  "event_type": "domain.entity.action",
  "timestamp": "ISO-8601",
  "data": {
    "field_1": "type",
    "field_2": "type"
  }
}
```

**Retry Policy**: _Max retries, backoff strategy_

**Dead Letter**: _Where failed events go_
