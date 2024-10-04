---
title: HTTP Service for Weather Info using Scala's Typelevel Stack
subtitle: Sample HTTP Service written in Scala using Typelevel stack which presents weather info based on longitude and latitude.
date: 2024-10-03T23:05:45-07:00
slug: 362993d
draft: false
author:
  name: Unnsse Khan
  link: https://github.com/unnsse
  email: contact@unnsse.io
  avatar:
description: Sample HTTP Service written in Scala using Typelevel stack which presents weather info based on longitude and latitude.
keywords: scala, typelevel, http4s, cats, cats-effect
license:
comment: false
weight: 0
tags:
  - scala, typelevel, http4s, cats, cats-effect
categories:
  - scala, typelevel
hiddenFromHomePage: false
hiddenFromSearch: false
hiddenFromRss: false
hiddenFromRelated: false
summary:
resources:
  - name: featured-image
    src: featured-image.jpg
  - name: featured-image-preview
    src: featured-image-preview.jpg
toc: false
math: false
lightgallery: false
password:
message:
repost:
  enable: true
  url:

# See details front matter: https://fixit.lruihao.cn/documentation/content-management/introduction/#front-matter
---

Just created a sample HTTP Service using Scala's [Typelevel](https://typelevel.org/) ([http4s](https://http4s.org/), [cats](https://typelevel.org/cats/), [cats-effect](https://typelevel.org/cats-effect/),[circe](https://circe.github.io/circe/), etc) stack 
which returns specific real-time weather information, based on the latitude and longitude parameters, passed into an HTTP Get endpoint.

Acceptance Criteria:

1. Accepts latitude and longitude coordinates.
2. Returns the short forecast for that area for Today (“Partly Cloudy” etc).
3. Categorizes whether the temperature is “hot”, “cold”, or “moderate”.
4. Uses the [National Weather Service API Web Service](https://www.weather.gov/documentation/services-web-api) as a data source.

This article serves as a high level overview of how to use some of Typelevel's projects to setup a working semi production ready HTTP Service which parses JSON and is testable in Scala.

---

The interesting mix of Typelevel's `http4s`, `cats`, and `cats-effect` libraries proved a different experience (and yet
very rewarding experience) due to the functional programming style and mindset enforced. This contrasts immensely with the 
traditional Java Spring Boot based Microservices codebases for which I have quite extensive professional experience
designing & developing in.

Used a top level singleton object to contain every method and expression which extend `cats.effect.IOApp`:

```scala
object WeatherServer extends IOApp { 
   // All the magic happens in here
}
```

To setup as an embedded HTTP service with a specific HTTP Get endpoint, I used `http4s` DSL convention for the routing:

```scala
// Define the weather service route
 val weatherService: HttpRoutes[IO] = HttpRoutes.of[IO] {
   case GET -> Root / "weather" :? LatitudeQueryParamMatcher(maybeLat) +& LongitudeQueryParamMatcher(maybeLon) =>
     (maybeLat, maybeLon) match {
       case (Some(lat), Some(lon)) if isValidCoordinate(lat, lon) =>
         getWeather(lat, lon).flatMap { weather =>
           Ok(weather.asJson)
         }
       case (Some(_), Some(_)) =>
         BadRequest("Invalid coordinates. Latitude must be between -90 and 90, and longitude must be between -180 and 180.")
       case _ =>
         BadRequest("Missing or invalid latitude and/or longitude parameters.")
     }
 }
```

In order, to handle the edge case of invalid longitude & latitude coordinates, created the following helper method:

```scala
// Validate coordinate range
private def isValidCoordinate(lat: Double, lon: Double): Boolean = {
  lat >= -90 && lat <= 90 && lon >= -180 && lon <= 180
}
```

Wrote the following method to categorize the temparture as cold, moderate or hot:

```scala
// Classify temperature into cold, moderate, or hot
def classifyTemperature(temp: Double): String = {
  if (temp < 60) "cold"
  else if (temp > 80) "hot"
  else "moderate"
}
```

Used the embedded `EmberServerBuilder` to server this HTTP Get endpoint on port 8080:

```scala
// Build and run the server
override def run(args: List[String]): IO[ExitCode] = {
  for {
    _ <- Logger[IO].info("Starting Weather Server")
    _ <- EmberServerBuilder
    .default[IO]
    .withHost(Host.fromString("0.0.0.0").get)
    .withPort(Port.fromInt(8080).get)
    .withHttpApp(weatherService.orNotFound)
    .build
    .useForever
  } yield ExitCode.Success
}
```

Also, good software engineering never neglects weeding out all the edge cases through a proper unit / integration test!

A combination of `cats.effect.IO` and `http4s` inside my `scalatest` helped maked this test coverage not only comprehensive but asynchronous:

```scala
package weather

import cats.effect.IO
import cats.effect.testing.scalatest.AsyncIOSpec
import org.http4s._
import org.http4s.implicits._
import org.scalatest.freespec.AsyncFreeSpec
import org.scalatest.matchers.should.Matchers
import io.circe.parser._
import org.typelevel.log4cats.Logger
import org.typelevel.log4cats.slf4j.Slf4jLogger

class WeatherServerSpec extends AsyncFreeSpec with AsyncIOSpec with Matchers {

  implicit val logger: Logger[IO] = Slf4jLogger.getLogger[IO]

  "WeatherServer" - {
    "classifyTemperature" - {
      "should classify temperatures correctly" in {
        WeatherServer.classifyTemperature(50) shouldBe "cold"
        WeatherServer.classifyTemperature(70) shouldBe "moderate"
        WeatherServer.classifyTemperature(90) shouldBe "hot"
      }
    }

    "getWeather" - {
      "should return weather response for valid coordinates" in {
        val result = WeatherServer.getWeather(37.7749, -122.4194)
        result.asserting { response =>
          response.forecast should not be empty
          Set("cold", "moderate", "hot") should contain(response.temperatureType)
        }
      }
    }
  }
  
  // More test cases covered can be found in Github repo source listing (see below)
}
```

Now, when  invoking with `sbt test`, seeing the tests passing was very gratifying (after jumping through a bunch of rabbit 
holes, not only for the acceptance criteria but dealing with the edge/corner cases that can arise) as this was my first foray 
using the Typelevel stack for working prototype:

```bash
23:58:15.779 [io-compute-7] INFO  weather.WeatherServer - Processed weather response: WeatherResponse(Mostly Clear,moderate)
[info] WeatherServerSpec:
[info] WeatherServer
[info]   classifyTemperature
[info]   - should classify temperatures correctly
[info]   getWeather
[info]   - should return weather response for valid coordinates
[info]   weatherService
[info]   - should handle valid requests
[info]   - should return 400 Bad Request for invalid coordinates
[info]   - should return 400 Bad Request for missing coordinates
[info] Run completed in 1 second, 407 milliseconds.
[info] Total number of tests run: 5
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 5, failed 0, canceled 0, ignored 0, pending 0
[info] All tests passed.
[success] Total time: 2 s, completed Oct 3, 2024, 11:58:16PM
```

To run locally, invoke `sbt run`:

```bash
[info] welcome to sbt 1.10.2 (Oracle Corporation Java 21)
[info] loading settings for project weatherservice-build-build from metals.sbt ...
[info] loading project definition from /Users/unnsse/work/WeatherService/project/project
[info] loading settings for project weatherservice-build from metals.sbt ...
[info] loading project definition from /Users/unnsse/work/WeatherService/project
[info] loading settings for project weatherservice from build.sbt ...
[info] set current project to weatherservice (in build file:/Users/unnsse/work/WeatherService/)
[info] running (fork) weather.WeatherServer 
[info] 00:40:28.776 [io-compute-5] INFO  weather.WeatherServer - Starting Weather Server
[info] 00:40:29.067 [io-compute-0] INFO  o.h.e.s.EmberServerBuilderCompanionPlatform - Ember-Server service bound to address: [::]:8080
```

With the HTTP Service running locally, one can hit the HTTP Get endpoint passing in longitude and latitude for San Francisco:

`http://localhost:8080/weather?latitude=37.7749&longitude=-122.4194`

Which will return the following real-time weather information in JSON format:

```json
{
    "forecast": "Mostly Clear",
    "temperatureType": "moderate"
}
```
---

This was a good learning experience how Scala's Typelevel stack can be used to create an HTTP API which sends requests 
to external data sources, parses JSON, and returns asynchronous responses.

The full code is available here: https://github.com/unnsse/WeatherService

Note: By no means, is this code production ready or even should be considered idiomatic Scala, this was just a mere exploratory
exercise.