---
layout: page
title: "Case study: migracja pipeline’ów CI na GitHub Actions"
---

## Kontekst

[krótko: jaki system, jaki problem biznesowy / techniczny]

## Cel

- skrócenie czasu buildów z X do Y
- zmniejszenie kosztów runnerów o Z%
- poprawa niezawodności CI

## Podejście

1. ...
2. ...

## Fragmenty kodu

```yaml
# .github/workflows/build-and-test.yml (wycinek)
name: Build and test

on:
  pull_request:
  push:
    branches: [ main ]

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
      - run: pytest
