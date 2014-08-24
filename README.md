NDN-JS:  A JavaScript client library for Named Data Networking
==============================================================

NDN-JS is the first native version of the NDN protocol written in JavaScript.  It
implements the [NDN-TLV wire format](http://named-data.net/doc/ndn-tlv/tlv.html) and is also wire format compatible with [PARC's CCNx](http://www.ccnx.org/releases/latest/doc/technical/CCNxProtocol.html).

The project by the UCLA NDN team - for more information on NDN, see http://named-data.net/.

See the file [INSTALL](https://github.com/named-data/ndn-js/blob/master/INSTALL) for build and installation instructions.

License
-------

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Lesser General Public License as published by
the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License
along with this program.  If not, see <http://www.gnu.org/licenses/>.
A copy of the GNU General Public License is in the file COPYING.

Overview
--------

This is a young project, with minimal documentation that we are slowly enhancing.  Please
submit any bugs or issues to the [NDN-JS issue tracker](http://redmine.named-data.net/projects/ndn-js/issues).

The primary goal of NDN-JS is to provide a pure JavaScript implementation of the NDN API
that enables developers to create browser-based applications using Named Data Networking.
The approach requires no native code or signed Java applets, and thus can be delivered
over the current web to modern browsers with no hassle for the end user.

Additional goals for the project:
- WebSockets transport (rather than TCP or UDP, which are not directly supported in
JavaScript).
- Relatively lightweight and compact, to enable efficient use on the web.
- Implement the [NDN-TLV wire format](http://named-data.net/doc/ndn-tlv/tlv.html) and be wire format compatible with [PARC's CCNx implementation](http://www.ccnx.org/releases/latest/doc/technical/CCNxProtocol.html) of NDN.

The library currently requires a remote NDN daemon, and has been tested with:

- ndnd from the [NDNx package](https://github.com/named-data/ndnx)
- ndnd-tlv from the [ndnd-tlv package](https://github.com/named-data/ndnd-tlv)
- NFD from the [NFD package](https://github.com/named-data/NFD).

Currently, the library has two APIs for developers:

1. The JavaScript API for asynchronous Interest/Data exchange.
   This uses WebSockets for transport and currently requires a
   proxy for communication with a remote ndnd daemon.

2. A Firefox plug-in, which implements an "ndn:/" url scheme
   following NDNx repository conventions for file retrieval.

By default, both parts of the library connect automatically to a set of
proxies and hubs that are part of the NDN research project's
[testbed](http://named-data.net/ndn-testbed/). There are currently no restrictions on
non-commercial, research-oriented data exchange on this testbed. See the
[Policies for Connecting Nodes to the NDN
Testbed](http://named-data.net/ndn-testbed/policies-connecting-nodes-ndn-testbed/),
or contact jburke@remap.ucla.edu for more details.

The developer can also
specify a local or remote ndnd as well, as an argument to the NDN constructor.

JavaScript API
--------------

See files in js/ and examples in tests/, examples/

NDN-JS currently supports expressing Interests (and receiving data) and publishing Data
(that answers Interests).  This includes encoding and decoding data packets as well as
signing and verifying them using RSA keys.

## NDN connectivity

The only way (for now) to get connectivity to other NDN nodes is via ndnd.  For the
JavaScript API, a WebSockets proxy that can communicate the target ndnd is currently
required.  Code for such a proxy (using Node.js) is in the wsproxy directory.  It
currently listens on port 9696 and passes messages (using either TCP or UDP) to ndnd on
the same host.

## Including the scripts in a web page

To use NDN-JS in a web page, one of two scripts must be included using a script tag:

ndn.js is a combined library (and ndn-js.min.js is a compressed version of the combined library).
See INSTALL for instructions on how to build these files.

Or they can be downloaded from the `build` directory:

- https://github.com/named-data/ndn-js/raw/master/build/ndn.js
- https://github.com/named-data/ndn-js/raw/master/build/ndn.min.js

## Examples

### ndn-ping

You can check out `examples/ndnping/ndn-ping.html` to see an example how to implement ndn-ping in NDN.js

### Example to retrieve content

A simple example of the current API to express an Interest and receive data:

    var face = new Face();  // connect to a default hub/proxy

    function onData(interest, data) {
      console.log("Received " + data.getName().toUri());
    }

    face.expressInterest(new Name("/ndn/edu/ucla/remap/ndn-js-test/hello.txt"), onData);

### Example to publish content

> Note that publishing content requires knowledge of a
> routable prefix for your upstream ndnd.  We are working
> on a way to either obtain that prefix or use the /local
> convention.

For now, see [tests/browser/test-publish-async.html].

Firefox add-on for the NDN protocol
-----------------------------------

See files in [ndn-protocol/](https://github.com/named-data/ndn-js/tree/master/ndn-protocol).

NDN-JS includes a Firefox extension for the ndn protocol built using the JavaScript
library.   It currently obtains NDN connectivity through the NDN testbed, but you can
click Set on the NDN Toolbar to change the connected hub.

To install, either download https://github.com/named-data/ndn-js/raw/master/ndn-protocol.xpi or use `ndn-protocol.xpi` in the distribution.

In Firefox, open `Tools > Add-ons`.  In the "gear" or "wrench" menu, click
`Install Add-on From File` and open `ndn-protocol.xpi`.  (In Firefox for Android,
type `file:` in the address bar and click the downloaded `ndn-protocol.xpi`.)
Restart Firefox.

Firefox uses the protocol extension to load any URI starting with ndn.  See this test page for examples:
ndn:/ndn/edu/ucla/remap/ndn-js-test/NDNProtocolExamples.html/?ndn.ChildSelector=1

When the page is loaded, Firefox updates the address bar with the full matched name from
the retrieved content object including the version, but without the implicit digest or
segment number (see below).

### Interest selectors in the ndn protocol:

You can add interest selectors. For example, this uses 1 to select the "rightmost" child
(latest version):
ndn:/ndn/edu/ucla/remap/ndn-js-test/howdy.txt?my=query&ndn.ChildSelector=1&key=value#ref

The browser loads the latest version and changes the address to:
ndn:/ndn/edu/ucla/remap/ndn-js-test/howdy.txt/%FD%052%A1%EA_%89?my=query&key=value#ref

The child selector was used and removed. Note that the other non-ndn query and
ref "?key=value#ref" are still present, in case they are needed by the web application.

The following selector keys are supported:

    ndn.MinSuffixComponent= non-negative int
    ndn.MaxSuffixComponents= non-negative int
    ndn.ChildSelector= non-negative int
    ndn.AnswerOriginKind= non-negative int
    ndn.Scope= non-negative int
    ndn.InterestLifetime= non-negative int (milliseconds)
    ndn.PublisherPublicKeyDigest= % escaped value
    ndn.Nonce= % escaped value
    ndn.Exclude= comma-separated list of % escaped values or * for ANY

### Multiple segments in the ndn protocol

A URI for content with multiple segments is handled as follows. If the URI has a segment
number, just retrieve that segment and return the content to the browser.

Otherwise look at the name in the returned ContentObject.  If the returned
name has no segment number, just return the content to the browser. If the
name has a segment number which is not 0, store it and express an interest for
segment 0. Also express an interest for the highest segment to try to
determine the FinalBlockID early. Fetch multiple segments in order and return
each content to the browser (in order) as the arrive until we get the segment
for FinalBlockID.
