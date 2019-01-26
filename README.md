# API Problem

[![License][license-image]][license-url] [![version][npm-image]][npm-url] [![Build Status][circle-image]][circle-url]

> [RFC 7807 - Problem Details for HTTP APIs](https://tools.ietf.org/html/rfc7807) Utility

## Install

```bash
npm install api-problem
```

## API

### Constructor: `Problem(status[, title][, type][, members])`

| name          | type     | required | default             | description                                                                            | referece                |
| ------------- | -------- | -------- | ------------------- | -------------------------------------------------------------------------------------- | ----------------------- |
| **`status`**  | `String` | `✔`      | `N/A`               | The HTTP status code generated by the origin server for this occurrence of the problem | [Section 3.1][spec-3.1] |
| **`title`**   | `String` | `✖`      | HTTP status phrase  | A short, human-readable summary of the problem type                                    | [Section 3.1][spec-3.1] |
| **`type`**    | `String` | `✖`      | `about:blank`       | A URI reference that identifies the problem type                                       | [Section 3.1][spec-3.1] |
| **`details`** | `Object` | `✖`      | `N/A`               | additional details to attach to object                                                 | [Section 3.1][spec-3.2] |

```js
import Problem from 'api-problem'

// HTTP defaults
new Problem(404)
//=> { status: '404', title: 'Not Found', type: 'http://www.iana.org/assignments/http-status-codes#404' }


// override defaults
new Problem(404, 'Oops! Page Not Found')
//=> { status: '404', title: 'Oops! Page Not Found', type: 'http://www.iana.org/assignments/http-status-codes#404' }


// custom values
new Problem(403, 'You do not have enough credit', 'https://example.com/probs/out-of-credit')
//=> { status: '403', title: 'You do not have enough credit', type: 'https://example.com/probs/out-of-credit' }


// additional details
new Problem(403, 'You do not have enough credit', 'https://example.com/probs/out-of-credit', {
  detail: 'Your current balance is 30, but that costs 50.',
  instance: '/account/12345/msgs/abc',
  balance: 30,
  accounts: ['/account/12345', '/account/67890']
})

//=> { status: '403', title: 'You do not have enough credit', type: 'https://example.com/probs/out-of-credit', detail: 'Your current balance is 30, but that costs 50.', instance: '/account/12345/msgs/abc', balance: 30, accounts: ['/account/12345', '/account/67890'] }


// HTTP defaults + Details
new Problem(403, {
  detail: 'Account suspended',
  instance: '/account/12345',
  date: '2016-01-15T06:47:01.175Z',
  account_id: '12345'
})
//=> { status: '403', title: 'Forbidden', type: 'http://www.iana.org/assignments/http-status-codes#404', detail: 'Account suspended', instance: '/account/12345', account_id: 12345, 'date: 2016-01-15T06:47:01.175Z' }

```

### Method : <string> `toString()`

returns a simplified, human-readable string representation

```js
let prob = new Problem(403, 'You do not have enough credit', 'https://example.com/probs/out-of-credit')

prob.toString() //=> [403] You do not have enough credit ('https://example.com/probs/out-of-credit')
```

### Method : <void> `send(response)`

uses [`response.writeHead`](https://nodejs.org/docs/latest/api/http.html#http_response_writehead_statuscode_statusmessage_headers) and [`response.end`](https://nodejs.org/docs/latest/api/http.html#http_response_end_data_encoding_callback) to send an appropriate error respnse message with the `Content-Type` response header to [`application/problem+json`][spec-3]

```js
import http from 'http'
import Problem from 'api-problem'

let response = new http.ServerResponse()
Problem.send(response)
```

### Express Middleware

A standard connect middleware is provided. The middleware intercepts Problem objects thrown as exceptions and serializes them appropriately in the HTTP response while setting the `Content-Type` response header to [`application/problem+json`][spec-3]

```js
import express from 'express'
import Problem from 'api-problem'
import Middleware from 'api-problem/lib/middleware'

let app = express()

app.get('/', (req, res) => {
  throw new Problem(403)
})

app.use(Middleware)
```

---
> Author: [Ahmad Nassri](https://www.ahmadnassri.com) &bull; 
> Github: [@AhmadNassri](https://github.com/ahmadnassri) &bull; 
> Twitter: [@AhmadNassri](https://twitter.com/ahmadnassri)

[license-url]: LICENSE
[license-image]: https://img.shields.io/github/license/ahmadnassri/node-api-problem.svg?style=for-the-badge&logo=circleci

[circle-url]: https://circleci.com/gh/ahmadnassri/workflows/node-api-problem
[circle-image]: https://img.shields.io/circleci/project/github/ahmadnassri/node-api-problem/master.svg?style=for-the-badge&logo=circleci

[npm-url]: https://www.npmjs.com/package/api-problem
[npm-image]: https://img.shields.io/npm/v/api-problem.svg?style=for-the-badge&logo=npm

[spec-3]: https://tools.ietf.org/html/rfc7807#section-3
[spec-3.1]: https://tools.ietf.org/html/rfc7807#section-3.1
[spec-3.2]: https://tools.ietf.org/html/rfc7807#section-3.2
