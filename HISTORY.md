# Changes

## Version 1.0.0 - 03/02/2013

 * Initial release

## Version 1.0.1 - 04/02/2013

 * Minor corrections to the README.md
 * Add note to README.md about error handling
 * Timed out errors are now instances of the `ping.RequestTimedOutError`
   object

## Version 1.0.2 - 11/02/2013

 * The RequestTimedOutError class is not being exported

## Version 1.1.0 - 13/02/2013

 * Support IPv6

## Version 1.1.1 - 15/02/2013

 * The `ping.Session.close()` method was not undefining the sessions raw
   socket after closing
 * Return self from the `pingHost()` method to chain method calls 

## Version 1.1.2 - 04/03/2013

 * Use the `raw.Socket.pauseRecv()` and `raw.Socket.resumeRecv()` methods
   instead of closing a socket when there are no more outstanding requests

## Version 1.1.3 - 07/03/2013

 * Sessions were limited to sending 65535 ping requests

## Version 1.1.4 - 09/04/2013

 * Add the `packetSize` option to the `createSession()` method to specify how
   many bytes each ICMP echo request packet should be

## Version 1.1.5 - 17/05/2013

 * Incorrectly parsing ICMP error responses resulting in responses matching
   the wrong request
 * Use a unique session ID per instance of the `Session` class to identify
   requests and responses sent by a session
 * Added the (internal) `_debugRequest()` and `_debugResponse()` methods, and
   the `_debug` option to the `createSession()` method
 * Added example programs `ping-ttl.js` and `ping6-ttl.js`
 * Use MIT license instead of GPL

## Version 1.1.6 - 17/05/2013

 * Session IDs are now 2 bytes (previously 1 byte), and request IDs are also
   now 2 bytes (previously 3 bytes)
 * Each ICMP error response now has an associated error class (e.g. the
   `Time exceeded` response maps onto the `ping.TimeExceededError` class)
 * Call request callbacks with an error when there are no free request IDs
   because of too many outstanding requests

## Version 1.1.7 - 19/05/2013

 * Added the `traceRoute()` method
 * Added the `ttl` option parameter to the `createSession()` method, and
   updated the example programs `ping-ttl.js` and `ping6-ttl.js` to use it
 * Response callback for `pingHost()` now includes two instances of the
   `Date` class to specify when a request was sent and a response received

## Version 1.1.8 - 01/07/2013

 * Use `raw.Socket.createChecksum()` instead of automatic checksum generation

## Version 1.1.9 - 01/07/2013

 * Use `raw.Socket.writeChecksum()` instead of manually rendering checksums

## Version 1.1.10 - 02/04/2014

 * Echo requests sent by this module are processed like responses when sent to
   the `127.0.0.1` and `::1` addresses

## Version 1.1.11 - 12/08/2014

 * Cannot specify the `retries` parameter for the `Session` class as `0`
 * Added example program `ping-retries-0.js`

## Version 1.1.12 - 22/09/2015

 * Host repository on GitHub

## Version 1.2.0 - 29/02/2016

 * Wrong callback called in the `traceRoute()` method when a session ID cannot
   be generated
 * Renamed the optional `ttl` parameter to the `traceRoute()` method to
   `ttlOrOptions`, and it is still optional
 * Permit users to control the number of permitted hop timeouts for the
   `traceRoute()` method, added the `maxHopTimeouts` parameter to the
   `traceRoute()` methods options
 * The `traceRoute()` method can start its trace at a higher ttl, added the
   `startTtl` parameter to the `traceRoute()` methods options
 * The `_expandConstantObject()` function was declaring variables with global
   scope

## Version 1.2.1 - 14/07/2017

 * Document the `Socket.getSocket()` method

## Version 1.2.2 - 06/06/2018

 * Set NoSpaceships Ltd to be the owner and maintainer

## Version 1.2.3 - 07/06/2018

 * Remove redundant sections from README.md

## Version 1.2.4 - 08/04/2024

 * Fix deprecation warning for `new Buffer()`
