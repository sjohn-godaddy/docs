---

## overall estimate

- one-time cost:    $1500
- monthly cost:     $1500/month
- daily run time:   2hrs (etl), 24hrs (dms)
- s3 data usage:    6 TB/month
- private ip used:  12/day (for 2hrs)

---

## monthly cost breakdown

#### dms
- first-time
    - c5.4xlarge $1.23/hr x 14 days (336 hrs) = $400
    - c5.2xlarge $0.62/hr x 30 days (672 hrs) = $400
- monthly
    - c5.large   $0.15/hr x 30 days (720 hrs) = $100
  
#### etl
- first-time
    - ($0.44/hr x 2 hrs x 5 DPUs) x 5 jobs x 30 days = $660
    - add another $240 for smaller jobs              = $140
- monthly
    - same ($800)
  
#### s3
- 200 GB/day x 30 days = $300

---
