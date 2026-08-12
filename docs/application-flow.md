# Application Request Flow

This document describes how a request travels through the three-tier architecture.

## Frontend Request

When a user opens the application:

```text
Browser
   |
   v
Internet-facing ALB
   |
   v
Web Target Group
   |
   v
Nginx
   |
   v
React Frontend
```

---

## API Request

The React application sends API requests using:

```text
/api/transaction
```

Nginx acts as a reverse proxy and forwards `/api/*` requests to the Internal Application Load Balancer.

```text
React
   |
   | /api/transaction
   v
Nginx
   |
   v
Internal ALB
   |
   v
Application Target Group
   |
   v
Node.js :4000
```

React does not communicate directly with individual backend EC2 instances.

---

## Database Request

The Node.js application communicates with Amazon Aurora MySQL:

```text
Node.js
   |
   | TCP :3306
   v
Aurora MySQL
   |
   v
webappdb.transactions
```

---

# Complete Flow

A transaction created from the browser follows:

```text
Browser
   |
   v
Internet-facing ALB
   |
   v
Web EC2
React + Nginx
   |
   v
Internal ALB
   |
   v
App EC2
Node.js
   |
   v
Aurora MySQL
```

The response then travels back through the same architecture to the browser.