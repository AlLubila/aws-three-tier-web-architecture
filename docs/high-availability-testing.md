# High Availability Testing

High availability was validated by intentionally terminating EC2 instances managed by the Auto Scaling Groups.

## Web Tier Failure Test

Initial state:

```text
              Public ALB
              /        \
             /          \
        Web EC2 A     Web EC2 B
```

One Web EC2 instance was intentionally terminated.

### Observed behavior

1. The instance was removed from service.
2. The Application Load Balancer continued routing requests to the remaining healthy instance.
3. The Auto Scaling Group detected that capacity was below the desired value.
4. A replacement EC2 instance was automatically launched using the Web Launch Template.
5. The new instance registered with the Web Target Group.
6. The `/health` check succeeded.
7. The new instance entered service.

The application remained available during the test.

---

# Application Tier Failure Test

Initial state:

```text
              Internal ALB
              /          \
             /            \
        App EC2 A       App EC2 B
```

One backend instance managed by the Application Auto Scaling Group was intentionally terminated.

### Observed behavior

1. The Internal ALB stopped routing requests to the terminated instance.
2. API traffic continued through the remaining healthy backend.
3. Auto Scaling detected the missing capacity.
4. A replacement instance was launched.
5. systemd automatically started the Node.js backend.
6. The application responded successfully to `/health`.
7. The instance became healthy in the Target Group.

No manual application startup was required.

---

# Result

Both infrastructure tiers demonstrated automatic recovery from individual EC2 instance failures.

```text
Instance failure
       |
       v
Load Balancer removes unhealthy target
       |
       v
Remaining instance serves traffic
       |
       v
Auto Scaling launches replacement
       |
       v
Health check passes
       |
       v
Normal capacity restored
```