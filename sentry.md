### Conversations recommendations for error monitoring
- We run integration tests every 3 minutes. If we get 2 test failures within 10 minutes then an alert goes out through pagerduty.
- We also have cloudewatch alarms set up for our SQS dead letter queues, eks clusters, cpu utilization
- We send all error-level logs to cloudwatch via fluentd with SQS plugin.
  - we can then create metrics in cloudwatch using a custom lambda
  - an alarm that will fire if there are two many error logs within a given amount of time
- Once we get an alarm, and we have to investigate an application/service exception, all logs are in kibana for us
- [Slack discussion](https://godaddy.slack.com/archives/C02C6HHK8ES/p1641245367424400)

### Other teams' recommendations for error monitoring
- Usually just logs (either via CloudWatch or Elastic). Your error logs should be including full stack traces. 
- Make sure your logs are including correlation IDs and are JSON-formatted so that you can query on specific logging event properties. 
- If you're looking for tracing, consider instrumenting your code for APM and sending your APM traces to your ESSP cluster.
- The Elastic APM agent has some simple error tracking built-in, but nothing close to Sentry's de-duplication/notification settings.
- [Slack discussion](https://godaddy.slack.com/archives/CBVHY0WRK/p1641316847079000)

### Takeaways
- Sentry has great features (error grouping, 24hr/14day dashboard, stacktraces) that is hard to replicate via elastic. Let's work on fixing the slack integration.
- Sentry grouping of errors requires some work from developers. When logging from code, we should start using parameters instead of putting it all into one string. Don't do [this](https://github.com/gdcorp-enm/reamaze/blob/main/app/controllers/incoming_controller.rb#L1031). Instead put the value as a parameter.
- Explore sentry alternatives
  - We can use Elastic logs, but re:amaze logging is not logging all errors to elastic. This can be fixed. [Here's an example](https://github.com/gdcorp-enm/reamaze/blob/main/app/controllers/api/v1/base_controller.rb#L15) where we log to sentry, but not to the logs. 
  - We can explore using Elastic APM agent for error tracking
