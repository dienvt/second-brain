---
tags:
  - comvex
---
> If we dont do that i think it will probably cause some issues with our recovery and stack trace and custom errors we do in the logging

May I ask detail about those issues. I guess the issues is stacktrace which application will report to sentry right? Because if we return error the first element of the trace is not where the error happend. Another theory I guess is the casing error didn't work