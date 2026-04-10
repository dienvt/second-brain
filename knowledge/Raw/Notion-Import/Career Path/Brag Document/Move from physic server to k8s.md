**S**ituation: Senior Software Engineer.

**T**ask: Move service from physical environment to k8s env

**A**ction:

- Research about k8s env
    - Rollout mechanism: Health check, ready check
    - Graceful shutdown: [https://github.com/vothanhdien/go-graceful-shutdown](https://github.com/vothanhdien/go-graceful-shutdown)
    - Stateless
    - Cluster
    - Monitoring
- Compare with current service in physical env
- List down which aspect don’t work in k8s env:
    - File config
    - Do I need to determine which env
- Turning Dockerfile
- Java project is difficult cause: large? not maintain code? black code block.

**R**esult:

- Move success ~50 service to k8s (both java and Golang) include tracing and monitory.