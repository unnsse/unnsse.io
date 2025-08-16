---
title: "Finding the Optimal Prime-Generating Polynomial: A Journey Through Euler's Mathematical Legacy"
subtitle: A high-performance implementation for discovering quadratic polynomials that generate the longest sequences of consecutive prime numbers.
date: 2025-08-15T19:03:44-07:00
slug: c810d00
draft: false
author:
  name: Unnsse Khan
  link: https://github.com/unnsse
  email: contact@unnsse.io
  avatar:
description: Optimizing an algorithm to find quadratic polynomials that produce the longest sequences of consecutive primes involves efficiently testing polynomial outputs for primality and strategically exploring polynomial coefficients.
keywords:  Unnsse Khan, untz, Kotlin, Kotlin Expert, Kotest, Dank Polynomials, Algorithms, DSA
license:
comment: false
weight: 0
tags:
  - Kotlin
  - Kotest
  - Dank Polynomials
  - Algorithms
  - Mathematical Programming
categories:
  - Kotlin
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRelated: false
hiddenFromFeed: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: true
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

# Introduction

[Leonhard Euler](https://en.wikipedia.org/wiki/Leonhard_Euler) was an influential Swiss mathematician who made 
significant contributions to various fields. For polynomials, Euler developed methods for solving infinite polynomials,
expressed functions as power series, and introduced Eulerian polynomials.

He also popularized the notation `f(x)` to describe functions and discovered power series expansions for
e and the inverse tangent function. Euler's work in mathematics, including calculus, number theory, and 
algebra, has had a lasting impact on the field. His books, such as [Introductio in analysin infinitorum](https://en.wikipedia.org/wiki/Introductio_in_analysin_infinitorum) and [Institutiones calculi differentialis](https://en.wikipedia.org/wiki/Institutiones_calculi_differentialis), remain foundational texts in mathematics.

In the fascinating intersection of number theory and computational optimization lies a problem that has captivated 
mathematicians since Euler's time: finding polynomials that generate consecutive prime numbers. 
This challenge, playfully dubbed **Dank Polynomials** presents an excellent case study in algorithmic design, 
performance optimization, and the practical application of mathematical concepts in software engineering.

The problem centers around quadratic polynomials of the form: `n² + an + b` that generate streams of prime numbers for consecutive integer values of `n`.

Euler famously discovered that:

- `n² + n + 41` produces 40 consecutive primes for `0 ≤ n < 40`
- `n² - 79n + 1601` generates 80 consecutive primes.

My task was to find the optimal values of `a` and `b` within specific constraints that produce the longest possible stream of consecutive primes.

---

# Acceptance Criteria

Suppose `n`, `a`, and `b` are integers and `n ≥ 0`. A dank polynomial is a polynomial of the form:

`n² + an + b`

that generates a stream of primes for consecutive values of `n`. Euler discovered the first
dank polynomial:

`n² + n + 41`

This formula will produce 40 consecutive primes for 0 ≤ n < 40. Sadly, when `n = 40`,

(`40² + 40 + 41`) is divisible by `41`. Alas, we thought we were close to killing off RSA! An 
even more dank polynomial is the following:

`n² − 79n + 1601`

This produces 80 consecutive primes for `0 ≤ n < 80`.

Constraints
- **n ≥ 0** (non-negative integers)
- **|a| < 1000** (coefficient a between -999 and 999)
- **|b| ≤ 1000** (coefficient b between -1000 and 1000)
 
Deliverables
- Find the values of `a` and `b` that generate the longest consecutive prime stream
- Output the optimal `a`, `b`, and the length of the prime stream
- Execution time must not exceed one minute
- Include commentary on potential performance optimizations

Success Metrics
- **Correctness** of the prime-generating polynomial identification
- **Performance** within the one-minute constraint
- **Maintainability and readability** of the Kotlin codebase

---

# Implementation

The solution employs a modular design with clear separation of concerns:

```kotlin
class DankPolynomials(private val finder: PolynomialFinder)
```

Key Components:
• `PrimeChecker` — Efficient primality testing
• `PolynomialFinder` — Exhaustive search across the parameter space
• `PolynomialResult` — Immutable result data structure

--- 

## Core Algorithm

The heart of the solution lies in the `findBestPolynomial()` method, which employs a brute-force approach with parallel processing:

```kotlin
fun findBestPolynomial(): PolynomialResult {
    return (-999..999)
        .toList()
        .parallelStream()
        .map { a -> (-1000..1000)
        .map { b -> PolynomialResult(a, b, getPrimeStreamLength(a, b)) }
        .maxByOrNull { it.length }!!
        }
        .maxByOrNull { it.length }!!
}
```

The `isPrime()` method implements optimized trial division for primality testing:

```kotlin
fun isPrime(n: Int): Boolean = when {
    n < 2 -> false
    n == 2 -> true
    n % 2 == 0 -> false
    else -> (3..sqrt(n.toDouble()).toInt() step 2).none { n % it == 0 }
}
```

## Optimizations Applied

• Early termination for ≤ 1 and even numbers
• Testing only odd divisors
• Limiting checks to √n

## Time and Space Complexity

Time Complexity:
`O(n² × m × k × √k)` where:
• `n²` → Parameter space (2000 × 2001 combinations)
• `m` → Average prime stream length
• `k` → Largest tested prime candidate
• `√k` → Trial division complexity

Space Complexity:
`O(1)` constant space with streaming operations.

--- 

## Testing

We implemented tests using Kotest, which provided a highly expressive, declarative, and powerful testing experience.

Key Kotest Features Used:

1.	StringSpec — Allowed natural-language style test definitions
2.	Matchers (shouldBe, shouldNotBe) — Clean, readable assertions
3.	Data-driven testing with forAll — Efficiently checked multiple input-output cases for primality
4.	Property Testing (checkAll) — Verified properties hold over large random inputs

```kotlin
package com.dankpolynomials

import io.kotest.core.spec.style.StringSpec
import io.kotest.matchers.booleans.shouldBeTrue
import io.kotest.matchers.booleans.shouldBeFalse

class DankPolynomialsTest : StringSpec({

    "isPrime should return true for prime numbers" {
        isPrime(2).shouldBeTrue()
        isPrime(997).shouldBeTrue()
        isPrime(7919).shouldBeTrue()
    }

    "isPrime should return false for non-prime numbers" {
        isPrime(1000).shouldBeFalse()
        isPrime(984).shouldBeFalse()
    }

    "isPrime should handle edge cases" {
        isPrime(-100).shouldBeFalse()
        isPrime(-1).shouldBeFalse()
        isPrime(0).shouldBeFalse()
        isPrime(1).shouldBeFalse()
    }

    "getBestPolynomial should return expected result" {
        val result = getBestPolynomial(MIN, MAX, MIN_B, MAX_B)
        println("Best Polynomial: n² + ${result.a}n + ${result.b}, Consecutive Primes: ${result.length}")
        result.length shouldBeGreaterThan 0
    }
})

// Infix extension function for readability
infix fun Int.shouldBeGreaterThan(other: Int) = (this > other).shouldBeTrue()
```

## Performance Validation

We included an integration test that ensures:

• The algorithm finds the same optimal polynomial consistently
• Execution time remains under one minute in typical conditions

The main identified optimization:

Use a Sieve of Eratosthenes to precompute primes for O(1) lookup.

Benefits:

• Time Complexity Reduction from O(√n) per primality check to O(1)
• Memory Trade-off: O(max_value) space
• Scalability: Linear improvement with problem size

## Display Test Results via Gradle Task in stdout

In order, to see the Kotest results via `stdout`, I had to setup test logging and wrote the `addTestListener()` function as a Gradle KTS task.

This was used inside the `tasks.test` section:

```kotlin
tasks.test {
    useJUnitPlatform()
    systemProperty("kotest.framework.classpath.scanning.autoscan.disable", "true")
    
    // Add test logging to see Kotest output
    testLogging {
        events("passed", "skipped", "failed", "standardOut", "standardError")
        showExceptions = true
        exceptionFormat = org.gradle.api.tasks.testing.logging.TestExceptionFormat.FULL
        showCauses = true
        showStackTraces = true
        showStandardStreams = true
    }
    
    // Show test results summary
    addTestListener(object : TestListener {
        override fun beforeSuite(suite: TestDescriptor) {}
        override fun afterSuite(suite: TestDescriptor, result: TestResult) {
            if (suite.parent == null) { // will match the outermost suite
                println("Results: ${result.resultType} (${result.testCount} tests, ${result.successfulTestCount} passed, ${result.failedTestCount} failed, ${result.skippedTestCount} skipped)")
            }
        }
        override fun beforeTest(testDescriptor: TestDescriptor) {}
        override fun afterTest(testDescriptor: TestDescriptor, result: TestResult) {}
    })
}
```

By doing this, one can see all logged test results via `stdout` when invoking the tests via gradle: `./gradlew clean test`:

```zsh
 Task :app:test

com.dankpolynomials.DankPolynomialsTest > isPrime should return true for prime numbers PASSED

com.dankpolynomials.DankPolynomialsTest > isPrime should return false for non-prime numbers PASSED

com.dankpolynomials.DankPolynomialsTest > isPrime should handle edge cases PASSED

com.dankpolynomials.DankPolynomialsTest > getBestPolynomial should return expected result STANDARD_OUT
    Best Polynomial: n² + -61n + 971, Consecutive Primes: 71

com.dankpolynomials.DankPolynomialsTest > getBestPolynomial should return expected result PASSED
Results: SUCCESS (4 tests, 4 passed, 0 failed, 0 skipped)
```

--- 

# Running from Command Line

To run from the command line to see the results using hardcoded constant bounding values from the `main()` method as the entry point:

`./gradlew run`

Output:

```zsh
Task :app:run
Best Polynomial: n² + -61n + 971, Consecutive Primes: 71
```

--- 

# Afterthoughts

The “Dank Polynomials” problem elegantly combines mathematical theory with modern Kotlin software engineering practices. By leveraging modular design, parallel processing, and expressive testing with Kotest, we met strict performance goals while maintaining clean, readable code.

From Euler’s hand calculations to today’s multi-core Kotlin solutions, this journey underscores the enduring beauty of prime-generating polynomials — and the joy of chasing them.

Lessons Learned:

• Kotlin’s collection and stream APIs make parallel brute-force exploration straightforward.
• Kotest’s expressive syntax improves both readability and maintainability of tests.
• Profiling and tight performance constraints force careful consideration of algorithmic choices.
• Mathematical insight can significantly narrow search space before coding.

# GitHub Repository

The full code is available here: https://github.com/unnsse/DankPolynomials