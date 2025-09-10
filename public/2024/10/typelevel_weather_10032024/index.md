# HTTP Service for Weather Info Using Scala's Typelevel Stack


# Introduction
[Scala](https://www.scala-lang.org) is a general purpose statically typed language that seamlessly blends object-oriented and functional programming. 
Its expressive syntax and functional features, along with its JVM compatibility, makes it an attractive choice for 
developers. Although it has a steep learning curve, mastering Scala can lead to highly performant and stable software, 
especially in concurrent environments. All of this is what caught my eye many years ago when I first came across Scala.

## Scala & Typelevel Stack 
The [Typelevel](https://typelevel.org/) stack is a Scala based ecosystem/framework of modular abstractions that emphasize 
functional programming and [Category Theory](https://en.wikipedia.org/wiki/Category_theory).

This article shares my experience creating an HTTP service using the Typelevel stack, in particular, [http4s](https://http4s.org/), [cats](https://typelevel.org/cats/), [cats-effect](https://typelevel.org/cats-effect/), and [circe](https://circe.github.io/circe/).
The service retrieves real-time weather information based on latitude and longitude passed to an HTTP GET endpoint.
It also provides a high-level overview of how to leverage Typelevel projects to quickly set up a semi-production-ready 
JSON parsing HTTP service with integration tests and logging.

# Acceptance Criteria
The HTTP Weather Service should adhere to the following requirements:

1. Accepts latitude and longitude coordinates.
2. Returns the short forecast for that area for Today (“Partly Cloudy” etc).
3. Categorizes whether the temperature is “hot”, “cold”, or “moderate”.
4. Uses the [National Weather Service API Web Service](https://www.weather.gov/documentation/services-web-api) as a data source.
5. Comprehensive code coverage using `scalatest`.

# Implementation
The interesting mix of Typelevel's [http4s](https://http4s.org/), [Cats](https://typelevel.org/cats), and [Cats Effect](https://typelevel.org/cats-effect) libraries proved a different (and yet
a very rewarding) experience due to the functional programming style and mindset enforced. This contrasts immensely with 
the traditional Java Spring Boot based Microservices codebases for which I have quite extensive professional experience
designing & developing in. The asynchronous capabilities of `Cats Effect`, combined with the powerful functional 
abstractions in `Cats` (e.g. `Monads`, `Applicatives`, `Show`, `Traverse`, etc), have shown me how to fully leverage 
Scala with Typelevel for specific distributed backend use cases.

## Top-Level Singleton Object
Started out by using a top level singleton object aptly named `WeatherServer`, to contain every method and expression, 
which extends `cats.effect.IOApp`:

```scala
object WeatherServer extends IOApp { 
   // All the magic happens in here
}
```
## HTTP GET Endpoint
For the specific HTTP GET endpoint, I used the `http4s` library's DSL convention for the routing:

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
## Valid Coordinates
In order, to handle the edge case of invalid longitude & latitude coordinates, created the following helper method:

```scala
// Validate coordinate range
private def isValidCoordinate(lat: Double, lon: Double): Boolean = {
  lat >= -90 && lat <= 90 && lon >= -180 && lon <= 180
}
```
## Classify Temperature
Wrote the following helper method to categorize the temperature as cold, moderate or hot:

```scala
// Classify temperature into cold, moderate, or hot
def classifyTemperature(temp: Double): String = {
  if (temp < 60) "cold"
  else if (temp > 80) "hot"
  else "moderate"
}
```
## Embedded HTTP Server
Used the embedded `EmberServerBuilder` to serve this HTTP GET endpoint on port 8080:

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
# Compilation & Bootstrap
## Compile project using sbt
To compile the project, invoke `sbt compile`:

```bash
 sbt compile
[info] welcome to sbt 1.10.2 (Oracle Corporation Java 21)
[info] loading settings for project weatherservice-build-build from metals.sbt ...
[info] loading project definition from /Users/unnsse/work/WeatherService/project/project
[info] loading settings for project weatherservice-build from metals.sbt ...
[info] loading project definition from /Users/unnsse/work/WeatherService/project
[info] loading settings for project weatherservice from build.sbt ...
[info] set current project to weatherservice (in build file:/Users/unnsse/work/WeatherService/)
[info] Executing in batch mode. For better performance use sbt's shell
[success] Total time: 0 s, completed Oct 4, 2024, 4:35:11 PM
```

## Bootstrap/Run WeatherServer using sbt
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
## HTTP GET Endpoint
With the HTTP Service running locally, one can hit the HTTP GET endpoint passing in longitude and latitude for San Francisco:

`http://localhost:8080/weather?latitude=37.7749&longitude=-122.4194`

Which will return the following real-time weather information in JSON format:

```json
{
    "forecast": "Mostly Clear",
    "temperatureType": "moderate"
}
```

# Testing
Also, good software engineering never neglects weeding out all the edge cases through a proper unit/integration test!

A combination of `cats.effect.IO` and `http4s` inside my `scalatest` helped make this test coverage not only 
comprehensive but asynchronous:

## Using scalatest
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
## Running tests using sbt
Now, when  invoking with `sbt test`, seeing the tests passing was very gratifying (after jumping through a bunch of rabbit 
holes, not only for the acceptance criteria, but dealing with the edge/corner cases that arose) as this was my first foray 
using the Typelevel stack for creating a working prototype:

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
[success] Total time: 2 s, completed Oct 3, 2024, 11:58:16 PM
```

# Afterthoughts
This was a good learning experience how Scala's Typelevel stack can be used to create an HTTP API which sends requests
to external data sources, parses JSON, and returns asynchronous responses. Also, it demonstrates how to test Typelevel
code using the `scalatest` library.

Future design considerations:

* Refactor by extracting out the helper methods into a `WeatherInfo` trait and then
extended that with a singleton object (e.g. `RealTimeWeather` extends `WeatherInfo`) from `WeatherServer`. 

* Refactor by extracting out the marshalled `circe` response case classes inside the `WeatherServer` into a separate 
package namespace.

* Externalize the National Weather Service API URL to a configuration file.

* Handle edge case of `ForecastProperties` containing an empty list of `Period` instances using `Option[List[Period]]`.

* Rewrite this using the [Tagless Final](https://typelevel.org/blog/2018/05/09/tagless-final-streaming.html) design 
pattern.

An example of applying `Tagless Final`, using the [higher kind](https://typelevel.org/blog/2016/08/21/hkts-moving-forward.html) `F[_]`
type for the proposed `WeatherInfo` trait would resemble this ADT-specific DSL:

```scala
trait WeatherInfo[F[_]] {
  def isValidCoordinate(lat: Double, lon: Double): F[Boolean]
  def classifyTemperature(temp: Double): F[Option[String]]
}
```
By using the `F[_]` higher kind type, you allow the caller to decide which effect type to use (e.g., IO, Future, etc.), 
making the trait abstract over different computational effects. More on Tagless Final later... 

# GitHub Repository
The full code is available here: https://github.com/unnsse/WeatherService

Note: By no means, is this code production ready or even should be considered idiomatic Scala, this was just a mere 
exploratory exercise.

---

> Author: [Unnsse Khan](https://github.com/unnsse)  
> URL: https://unnsse.io/2024/10/typelevel_weather_10032024/  

