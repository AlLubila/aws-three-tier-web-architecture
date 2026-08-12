# Dynamic Auto Scaling Test

Dynamic scaling was configured and tested on the Web Tier.

## Configuration

| Setting | Value |
|---|---|
| Scaling policy | Target Tracking |
| Metric | Average CPU Utilization |
| Target value | 50% |
| Minimum capacity | 2 |
| Desired capacity | 2 |
| Maximum capacity | 4 |
| Instance warmup | 300 seconds |

---

# Test Method

CPU load was intentionally generated on Web EC2 instances using:

```bash
stress-ng --cpu $(nproc) --timeout 10m
```

This simulated CPU-intensive application workload.

---

# Scale-Out

CPU utilization increased above the configured target.

The following behavior was observed:

```text
CPU utilization increases
          |
          v
CloudWatch CPU metric
          |
          v
Target Tracking policy
          |
          v
Auto Scaling Group
          |
          v
Desired capacity increases
          |
          v
Launch Template creates EC2
          |
          v
Instance joins Target Group
          |
          v
Health check passes
          |
          v
ALB starts using the instance
```

The Web Tier successfully increased beyond its normal capacity of two instances.

---

# Scale-In

After the artificial CPU workload ended, utilization decreased.

Auto Scaling detected the excess capacity and automatically terminated additional instances.

The group returned toward its minimum capacity:

```text
4 instances
     |
CPU decreases
     |
     v
3 instances
     |
     v
2 instances
```

This validated both automatic scale-out and scale-in behavior.