# Project Conventions

## gegi-monolith: Running Tests

```bash
docker exec -e XDEBUG_MODE=coverage gegi-monolith-rcdemo.gegi.co ./ge.php run tests --tests=TestClassName.php
```

- Replace `TestClassName.php` with the actual test class name (e.g., `TuitionBillsConfirmTest.php`)
- Multiple tests: `--tests=Test1.php,Test2.php`

## Landing the Plane (Session Completion)

**MANDATORY WORKFLOW:**

1. **File issues for remaining work** - Create issues for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
