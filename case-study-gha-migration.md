---
layout: default
title: "Case study: migracja pipeline’ów CI na GitHub Actions"
permalink: /case-study-gha-migration.html
published: false
---

[← Back to index](/)

## Kontekst

[Krótko: jaki system, jaki był problem biznesowy / techniczny.]

## Cel

- skrócenie czasu buildów z X do Y,
- zmniejszenie kosztów runnerów o Z%,
- poprawa niezawodności CI.

## Podejście

1. Audyt istniejących pipeline’ów.
2. Zaprojektowanie docelowej architektury CI na GitHub Actions.
3. Migracja krok po kroku i wygaszanie starego rozwiązania.

## Fragmenty kodu

~~~yaml
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
~~~

## Rezultaty

- [Tu wypisz konkretne liczby / usprawnienia.]

## Lessons learned

- [Czego się nauczyłeś, co byś zrobił inaczej.]
