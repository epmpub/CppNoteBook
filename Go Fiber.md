# Go Fiber

The application itself is rather basic. It retrieves a name from a query string and then returns a salutation message.

## echo

```go
package main

import (
 "fmt"
 "net/http"

 "github.com/labstack/echo/v4"
)

func main() {
 e := echo.New()
 e.GET("/", func(c echo.Context) error {
  name := c.QueryParam("name")
  return c.String(http.StatusOK, fmt.Sprintf("Hello, %s", name))
 })

 e.Start(":8000")
}
```

## fiber

```go
package main

import (
 "fmt"

 "github.com/gofiber/fiber/v2"
)

func main() {
 app := fiber.New()
 app.Get("/", func(c *fiber.Ctx) error {
  name := c.Query("name")
  return c.SendString(fmt.Sprintf("Hello, %s", name))
 })

 app.Listen(":8000")
}
```

Upon examining the two sets of code, their similarities are striking with only a few minor distinctions present.

To gauge the performance, I used [Bombardier](https://github.com/codesenberg/bombardier) as my testing tool.

```
bombardier -c 125 -n 10000000 http://localhost:8000?name=leon
```

This executes 10,000,000 requests with 125 connections to the given url.

# Test Results

Here’s the outcome.

## echo web application

```
Statistics Avg Stdev Max
 Reqs/sec 135901.05 22176.79 171994.39
 Latency 0.92ms 400.54us 47.35ms
 HTTP codes:
 1xx - 0, 2xx - 10000000, 3xx - 0, 4xx - 0, 5xx - 0
 others - 0
 Throughput: 26.05MB/s
```

## fiber web application

```
Statistics Avg Stdev Max
 Reqs/sec 231584.74 22757.64 275259.17
 Latency 537.09us 2.42ms 324.19ms
 HTTP codes:
 1xx - 0, 2xx - 10000000, 3xx - 0, 4xx - 0, 5xx - 0
 others - 0
 Throughput: 44.40MB/s
```

When stacked against Echo, Fiber is observed to manage, on average, approximately 1.7 times more requests per second, peaking at 1.6 times more.

Nevertheless, when considering maximum latency, Fiber’s performance was significantly less impressive than Echo’s. The reason for this is unclear to me at this time. Perhaps, a deeper and more comprehensive exploration of the differences between Echo and Fiber is warranted.

# Conclusion

From this modest experiment, it can be asserted that Fiber is indeed faster — outpacing the Echo web framework. However, the simplicity of this test must be taken into account. To draw a more detailed comparison, this blog post shouldn’t be your only source of information as it only provides a brief glimpse into the comparative performance.