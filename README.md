
# net-ping

This module is a fork of [net-ping](https://github.com/nospaceships/node-net-ping).
Its dependency on [raw-socket](https://github.com/nospaceships/node-raw-socket) has been changed to [@neuralegion/raw-socket](https://github.com/NeuraLegion/node-raw-socket), which uses prebuildify to bundle platform builds. This reduces the need for local builds.

This module implements ICMP Echo (ping) support for [Node.js][nodejs].

This module is installed using [node package manager (npm)][npm]:

    npm install net-ping

It is loaded using the `require()` function:

    var ping = require ("net-ping");

A ping session can then be created to ping or trace route to many hosts:

    var session = ping.createSession ();

    session.pingHost (target, function (error, target) {
        if (error)
            console.log (target + ": " + error.toString ());
        else
            console.log (target + ": Alive");
    });

[nodejs]: http://nodejs.org "Node.js"
[npm]: https://npmjs.org/ "npm"

# Network Protocol Support

This module supports IPv4 using the ICMP, and IPv6 using the ICMPv6.

# Error Handling

Each request exposed by this module requires one or more mandatory callback
functions.  Callback functions are typically provided an `error` argument.

All errors are sub-classes of the `Error` class.  For timed out errors the
error passed to the callback function will be an instance of the
`ping.RequestTimedOutError` class, with the exposed `message` attribute set
to `Request timed out`.

This makes it easy to determine if a host responded, a time out occurred, or
whether an error response was received:

    session.pingHost ("1.2.3.4", function (error, target) {
        if (error)
            if (error instanceof ping.RequestTimedOutError)
                console.log (target + ": Not alive");
            else
                console.log (target + ": " + error.toString ());
        else
            console.log (target + ": Alive");
    });

In addition to the the `ping.RequestTimedOutError` class, the following errors
are also exported by this module to wrap ICMP error responses:

 * `DestinationUnreachableError`
 * `PacketTooBigError`
 * `ParameterProblemError`
 * `RedirectReceivedError`
 * `SourceQuenchError`
 * `TimeExceededError`

These errors are typically reported by hosts other than the intended target.
In all cases each class exposes a `source` attribute which will specify the
host who reported the error (which could be the intended target).  This will
also be included in the errors `message` attribute, i.e.:

    $ sudo node example/ping-ttl.js 1 192.168.2.10 192.168.2.20 192.168.2.30
    192.168.2.10: Alive
    192.168.2.20: TimeExceededError: Time exceeded (source=192.168.1.1)
    192.168.2.30: Not alive

The `Session` class will emit an `error` event for any other error not
directly associated with a request.  This is typically an instance of the
`Error` class with the errors `message` attribute specifying the reason.

# Packet Size

By default ICMP echo request packets sent by this module are 16 bytes in size.
Some implementations cannot cope with such small ICMP echo requests.  For
example, some implementations will return an ICMP echo reply, but will include
an incorrect ICMP checksum.

This module exposes a `packetSize` option to the `createSession()` method which
specifies how big ICMP echo request packets should be:

    var session = ping.createSession ({packetSize: 64});

# Round Trip Times

Some callbacks used by methods exposed by this module provide two instances of
the JavaScript `Date` class specifying when the first ping was sent for a
request, and when a request completed.

These parameters are typically named `sent` and `rcvd`, and are provided to
help round trip time calculation.

A request can complete in one of two ways.  In the first, a ping response is
received and `rcvd - sent` will yield the round trip time for the request in
milliseconds.

In the second, no ping response is received resulting in a request time out.
In this case `rcvd - sent` will yield the total time spent waiting for each
retry to timeout if any.  For example, if the `retries` option to the
`createSession()` method was specified as `2` and `timeout` as `2000` then
`rcvd - sent` will yield more than `6000` milliseconds.

Although this module provides instances of the `Date` class to help round trip
time calculation the dates and times represented in each instance should not be
considered 100% accurate.

Environmental conditions can affect when a date and time is actually
calculated, e.g. garbage collection introducing a delay or the receipt of many
packets at once.  There are also a number of functions through which received
packets must pass, which can also introduce a slight variable delay.

Throughout development experience has shown that, in general the smaller the
round trip time the less accurate it will be - but the information is still
useful nonetheless.

# Constants

The following sections describe constants exported and used by this module.

## ping.NetworkProtocol

This object contains constants which can be used for the `networkProtocol`
option to the `createSession()` function exposed by this module.  This option
specifies the IP protocol version to use when creating the raw socket.

The following constants are defined in this object:

 * `IPv4` - IPv4 protocol
 * `IPv6` - IPv6 protocol

# Using This Module

The `Session` class is used to issue ping and trace route requests to many
hosts.  This module exports the `createSession()` function which is used to
create instances of the `Session` class.

## ping.createSession ([options])

The `createSession()` function instantiates and returns an instance of the
`Session` class:

    // Default options
    var options = {
        networkProtocol: ping.NetworkProtocol.IPv4,
        packetSize: 16,
        retries: 1,
        sessionId: (process.pid % 65535),
        timeout: 2000,
        ttl: 128
    };
    
    var session = ping.createSession (options);

The optional `options` parameter is an object, and can contain the following
items:

 * `networkProtocol` - Either the constant `ping.NetworkProtocol.IPv4` or the
   constant `ping.NetworkProtocol.IPv6`, defaults to the constant
   `ping.NetworkProtocol.IPv4`
 * `packetSize` - How many bytes each ICMP echo request packet should be,
   defaults to `16`, if the value specified is less that `12` then the value
   `12` will be used (8 bytes are required for the ICMP packet itself, then 4
   bytes are required to encode a unique session ID in the request and response
   packets)
 * `retries` - Number of times to re-send a ping requests, defaults to `1`
 * `sessionId` - A unique ID used to identify request and response packets sent
   by this instance of the `Session` class, valid numbers are in the range of
   `1` to `65535`, defaults to the value of `process.pid % 65535`
 * `timeout` - Number of milliseconds to wait for a response before re-trying
   or failing, defaults to `2000`
 * `ttl` - Value to use for the IP header time to live field, defaults to `128`

After creating the ping `Session` object an underlying raw socket will be
created.  If the underlying raw socket cannot be opened an exception with be
thrown.  The error will be an instance of the `Error` class.

Seperate instances of the `Session` class must be created for IPv4 and IPv6.

## session.on ("close", callback)

The `close` event is emitted by the session when the underlying raw socket
is closed.

No arguments are passed to the callback.

The following example prints a message to the console when the underlying raw
socket is closed:

    session.on ("close", function () {
        console.log ("socket closed");
    });

## session.on ("error", callback)

The `error` event is emitted by the session when the underlying raw socket
emits an error.

The following arguments will be passed to the `callback` function:

 * `error` - An instance of the `Error` class, the exposed `message` attribute
   will contain a detailed error message.

The following example prints a message to the console when an error occurs
with the underlying raw socket, the session is then closed:

    session.on ("error", function (error) {
        console.log (error.toString ());
        session.close ();
    });

## session.close ()

The `close()` method closes the underlying raw socket, and cancels all
outstanding requsts.

The calback function for each outstanding ping requests will be called.  The
error parameter will be an instance of the `Error` class, and the `message`
attribute set to `Socket forcibly closed`.

The sessoin can be re-used simply by submitting more ping requests, a new raw
socket will be created to serve the new ping requests.  This is a way in which
to clear outstanding requests.

The following example submits a ping request and prints the target which
successfully responded first, and then closes the session which will clear the
other outstanding ping requests.

    var targets = ["1.1.1.1", "2.2.2.2", "3.3.3.3"];
    
    for (var i = 0; i < targets.length; i++) {
        session.pingHost (targets[i], function (error, target) {
            if (! error) {
                console.log (target);
                session.close (); 
            }
        });
    }

## session.getSocket ()

The `getSocket()` method returns the underlying raw socket used by the session.
This class is an instance of the `Socket` class exposed by the
[raw-socket][raw-socket] module.  This can be used to modify properties of the
raw socket, such as specifying which network interface ICMP messages should be
sent from.

In the following example the network interface from which to send ICMP messages
is set:

	var raw = require("raw-socket") // Required for access to constants

	var level  = raw.SocketLevel.SOL_SOCKET
	var option = raw.SocketOption.SO_BINDTODEVICE

	var iface  = Buffer.from("eth0")

	session.getSocket().setOption(level, option, iface, iface.length)

[raw-socket]: https://www.npmjs.com/package/raw-socket "raw-socket"

## session.pingHost (target, callback)

The `pingHost()` method sends a ping request to a remote host.

The `target` parameter is the dotted quad formatted IP address of the target
host for IPv4 sessions, or the compressed formatted IP address of the target
host for IPv6 sessions.

The `callback` function is called once the ping requests is complete.  The
following arguments will be passed to the `callback` function:

 * `error` - Instance of the `Error` class or a sub-class, or `null` if no
   error occurred
 * `target` - The target parameter as specified in the request
   still be the target host and NOT the responding gateway
 * `sent` - An instance of the `Date` class specifying when the first ping
   was sent for this request (refer to the Round Trip Time section for more
   information)
 * `rcvd` - An instance of the `Date` class specifying when the request
   completed (refer to the Round Trip Time section for more information)

The following example sends a ping request to a remote host:

    var target = "fe80::a00:27ff:fe2a:3427";

    session.pingHost (target, function (error, target, sent, rcvd) {
        var ms = rcvd - sent;
        if (error)
            console.log (target + ": " + error.toString ());
        else
            console.log (target + ": Alive (ms=" + ms + ")");
    });

## session.traceRoute (target, ttlOrOptions, feedCallback, doneCallback)

The `traceRoute()` method provides similar functionality to the trace route
utility typically provided with most networked operating systems.

The `target` parameter is the dotted quad formatted IP address of the target
host for IPv4 sessions, or the compressed formatted IP address of the target
host for IPv6 sessions.  The optional `ttlOrOptions` parameter can be either a
number which specifies the maximum number of hops used by the trace route,
which defaults to the `ttl` options parameter as defined by the
`createSession()` method, or an object which can contain the following
parameters:

 * `ttl` - The maximum number of hops used by the trace route, defaults to the
   `ttl` options parameter as defined by the `createSession()` method
 * `maxHopTimeouts` - The maximum number of hop timeouts that should occur,
   defaults to `3`
 * `startTtl` - Starting ttl for the trace route, defaults to `1`

Some hosts do not respond to ping requests when the time to live is `0`, that
is they will not send back an time exceeded error response.  Instead of
stopping the trace route at the first time out this method will move on to the
next hop, by increasing the time to live by 1.  It will do this 2 times by
default, meaning that a trace route will continue until the target host
responds or at most 3 request time outs are experienced.  The `maxHopTimeouts`
option above can be used to control how many hop timeouts can occur.

Each requst is subject to the `retries` and `timeout` option parameters to the
`createSession()` method.  That is, requests will be retried per hop as per
these parameters.

This method will not call a single callback once the trace route is complete.
Instead the `feedCallback` function will be called each time a ping response is
received or a time out occurs. The following arguments will be passed to the
`feedCallback` function:

 * `error` - Instance of the `Error` class or a sub-class, or `null` if no
   error occurred
 * `target` - The target parameter as specified in the request
 * `ttl` - The time to live used in the request which triggered this respinse
 * `sent` - An instance of the `Date` class specifying when the first ping
   was sent for this request (refer to the Round Trip Time section for more
   information)
 * `rcvd` - An instance of the `Date` class specifying when the request
   completed (refer to the Round Trip Time section for more information)

Once a ping response has been received from the target, or more than three
request timed out errors are experienced, the `doneCallback` function will be
called. The following arguments will be passed to the `doneCallback` function:

 * `error` - Instance of the `Error` class or a sub-class, or `null` if no
   error occurred
 * `target` - The target parameter as specified in the request

Once the `doneCallback` function has been called the request is complete and
the `requestCallback` function will no longer be called.

If the `feedCallback` function returns a true value when called the trace route
will stop and the `doneCallback` will be called.

The following example initiates a trace route to a remote host:

    function doneCb (error, target) {
        if (error)
            console.log (target + ": " + error.toString ());
        else
            console.log (target + ": Done");
    }

    function feedCb (error, target, ttl, sent, rcvd) {
        var ms = rcvd - sent;
        if (error) {
            if (error instanceof ping.TimeExceededError) {
                console.log (target + ": " + error.source + " (ttl="
                        + ttl + " ms=" + ms +")");
            } else {
                console.log (target + ": " + error.toString ()
                        + " (ttl=" + ttl + " ms=" + ms +")");
            }
        } else {
            console.log (target + ": " + target + " (ttl=" + ttl
                    + " ms=" + ms +")");
        }
    }

    session.traceRoute ("192.168.10.10", 10, feedCb, doneCb);

# Example Programs

Example programs are included under the modules `example` directory.
