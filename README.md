# _vroughtime_: a compact roughtime client

_vroughtime_ is a compact C library for `roughtime`, a secure Network Time Protocol (NTP) replacement,
targeting at embedded use. `roughtime` provides cryptographically authenticated time.
This is useful if your cryptographic protocol relies on time (eg certificate expiration times).

A `roughtime` client is relatively simple and lightweight to implement,
even in lower-end processors, since it does not use TLS.
`roughtime` is essentially a challenge-response protocol:
the client sends a random challenge to ensure freshness
and the server replies with a signed time tied to the
client's challenge.
For a nicer explanation on `roughtime`,
[look here](https://roughtime.googlesource.com/roughtime/#roughtime-1).
For the protocol specification,
[click here](https://roughtime.googlesource.com/roughtime/+/HEAD/PROTOCOL.md).

`vroughtime` is a portable implementation with no dependency on the transport layer.
You need to provide this.

## Usage

To make things easy, _vroughtime_ does not even ship with
a build system. You just need the following files:

* vrt.c
* vrt.h
* tweetnacl.c
* tweetnacl.h

Link against `vrt.o` and `tweetnacl.o`. There are no
other dependencies. To use,
you'll need the following buffers:

```
uint8_t query[VRT_QUERY_PACKET_LEN];
uint8_t nonce[VRT_NONCE_SIZE];
uint32_t recv_buffer[RECV_BUFFER_LEN / 4];
```

First prepare the query packet like:
```
vrt_ret_t ret;

fill_random(nonce, sizeof nonce); // you need to provide this
ret = vrt_make_query(nonce, 64, query, sizeof query)
fail_on(ret != VRT_SUCCESS);
```

Then send `query` using UDP to the roughtime server of your choice,
and store the answer in `recv_buffer`. Parse the response
as:

```
uint64_t out_midpoint;
uint32_t out_radii;
ret = vrt_parse_response(nonce, recv_buffer,
                         sizeof recv_buffer * sizeof recv_buffer[0],
                         public_key, &out_midpoint, &out_radii);
fail_on(ret != VRT_SUCCESS);
```

The output will be placed in `out_midpoint`. Check `vrt_test.c` and `vrt_client_unix.c` for an example on how to use it.

For a list of current roughtime servers, see
https://github.com/cloudflare/roughtime/blob/master/ecosystem.json

## Notes

A natural question is why use C in 2021 for code that parses untrusted input,
deals with tag-length-values and calls crypto. This looks like mixing water
with electricity. The answer is, esteemed reader, a little more nuanced to
fit in a sentence.

**Warning**: _vroughtime_ is the product of a single human with zero peer review.
Meaning that it will have bugs. So if you have some time to dispose of,
audit the code (the relevant bits are only about 200 lines) and leave
a PR with your findings (even if none!). And please add your name here ❤️

## Alternatives

For a more complete implementation, check out https://github.com/nahojkap/craggy
