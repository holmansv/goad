# HolmanSV Load Tester

Read more at 
<https://goad.io>

Goad is an AWS Lambda powered, highly distributed,
load testing tool built in Go for the 2016 [Gopher Gala][].

![Go + Load ⇒ Goad](https://goad.io/assets/go-plus-load.png)

Goad allows you to load test your websites from all over the world whilst costing you the tiniest fractions of a penny by using AWS Lambda in multiple regions simultaneously.

## How it works

Goad takes full advantage of the power of Amazon Lambdas and Go's concurrency for distributed load testing. You can use Goad to launch HTTP loads from up to four AWS regions at once. Each lambda can handle hundreds of concurrent connections, we estimate that Goad should be able to achieve peak loads of up to **100,000 concurrent requests**.

![Goad diagram](https://goad.io/assets/diagram.svg)

Running Goad will create the following AWS resources:

- An IAM Role for the lambda function.
- An IAM Role Policy that allows the lambda function to send messages to SQS, to publish logs and to spawn new lambda in case an individual lambda times out on a long running test.
- A lambda function.
- An SQS queue for the test.

A new SQS queue is created for each test run, and automatically deleted after the test is completed. The other AWS resources are reused in subsequent tests.

Flags:


### Docker

Goad can also be run as a Docker container which exposes the web API:

    docker build -t goad .
    docker run --rm -p 8080:8080 -e AWS_ACCESS_KEY_ID=<your key ID> -e AWS_SECRET_ACCESS_KEY=<your key> goad

You can then execute a load test using WebSocket:

    wsc 'ws://localhost:8080/load?url=<Your URL>&requests=1000&concurrency=10&timelimit=3600&timeout=15&region[]=us-east-1&region[]=eu-west-1&header[]=Authorization: Bearer <Your JWT>&method=POST&body={"hello":"world"}&header[]=Content-Type: application/json&header[]=Accept: application/json'
    
```
  requests=1000            Number of requests to perform. Set to 0 in combination with a specified timelimit allows for unlimited requests for the specified time.
  concurrency=10           Number of multiple requests to make at a time
  timelimit=3600           Seconds to max. to spend on benchmarking
  timeout=15               Seconds to max. wait for each response
  header=HEADER ...        Add Arbitrary header line, eg. 'Accept-Encoding: gzip' (repeatable)
  region=us-east-1... ...  AWS regions to run in. Repeat flag to run in more then one region. (repeatable)
  method=GET               HTTP method
  body=BODY                HTTP request body
   
    
 'requests': 1000

 'concurrency': 10

 'timelimit': 3600

 'timeout': 15

 'region[]'[0]: us-east-1

 'region[]'[1]: eu-west-1

 'header[]'[0]: Authorization: Bearer <Your JWT>

 'header[]'[1]: Content-Type: application/json

 'header[]'[2]: Accept: application/json

 'method': POST

 'body': {"hello":"world"}    
```

## How it was built

### Go CLI and server

* [AWS SDK for Go][]
* [Gorilla WebSocket][]
* [Termbox][]
* [UUID][]
* [bindata][]

### Goad executable

Written in pure Go, Goad takes care of instantiating all the AWS resources, collecting results and displaying them. Interestingly, it contains the executable of the Lambda worker, which is also written in Go.

There is also a webapi version, which can be used to serve Goad as a web service. This streams the results using WebSockets.

### Lambda workers

AWS Lambda instances are bootstrapped using node.js but the actual work on the Lambda instances is performed by a Go process. The HTTP
requests are distributed among multiple Lambda instances each running multiple concurrent goroutines, in order to achieve the desired
concurrency level with high throughput.

## License & Copyright

MIT License. Copyright 2016 [Joao Cardoso][], [Matias Korhonen][], [Rasmus Sten][], and [Stephen Sykes][].

See the LICENSE file for more details.

[Goad.io]: https://goad.io
[GitHub Releases]: https://github.com/gophergala2016/goad/releases

[AWS SDK for Go]: http://aws.amazon.com/sdk-for-go/
[Gorilla WebSocket]: https://github.com/gorilla/websocket
[Termbox]: https://github.com/nsf/termbox-go
[UUID]: https://github.com/satori/go.uuid
[bindata]: https://github.com/jteeuwen/go-bindata
[toml]: https://github.com/toml-lang/toml

[Gopher Gala]: http://gophergala.com/
[Joao Cardoso]: https://twitter.com/jcxplorer
[Matias Korhonen]: https://twitter.com/matiaskorhonen
[Rasmus Sten]: https://twitter.com/pajp
[Stephen Sykes]: https://twitter.com/sdsykes
