# MessagePack Erlang

[![Travis](https://secure.travis-ci.org/msgpack/msgpack-erlang.png)](https://travis-ci.org/msgpack/msgpack-erlang)

[![Drone.io](https://drone.io/github.com/msgpack/msgpack-erlang/status.png)](https://drone.io/github.com/msgpack/msgpack-erlang)

## prequisites for runtime

[Erlang/OTP](http://erlang.org/), >= 17.0
Also based on [the new msgpack spec 232a0d](https://github.com/msgpack/msgpack/blob/232a0d14c6057000cc4a478f0dfbb5942ac54e9e/spec.md).

## edit rebar.config to use in your application

```erlang
{deps, [
  {msgpack, ".*",
    {git, "git://github.com/msgpack/msgpack-erlang.git", {branch, "master"}}}
]}.
```

## Simple deserialization

```erlang
Ham = msgpack:pack(Spam),
{ok, Spam} = msgpack:unpack(Ham).
```

## Stream deserialization

```erlang
{Term0, Rest0} = msgpack:unpack_stream(Binary),
{Term1, Rest1} = msgpack:unpack_stream(Rest0),
...
```

## String type

Now this supports string type!

```erlang
Opt = [{enable_str, true}]
{ok, "埼玉"} = msgpack:unpack(msgpack:pack("埼玉", Opt), Opt).
 => {ok,[22524,29577]}
```

There are several options for `msgpack:pack/2` and `msgpack:unpack/2` .
See `msgpack:options()` in `msgpack.hrl`.

## Map Style

Since Erlang/OTP 17.0

```erlang
msgpack:pack(#{ <<"key">> => <<"value">> }, [{format, map}]).
```

Or use old jiffy/jsx style

```erlang
msgpack:pack({[{<<"key">>, <<"value">>}]}, [{format, jiffy}]),
msgpack:pack([{<<"key">>, <<"value">>}], [{format, jsx}]).
```

## Ext type

Now msgpack-erlang supports ext type. Now you can serialize everything
with your original (de)serializer. That will enable us to handle
erlang- native types like `pid()`, `ref()` contained in `tuple()`. See
`test/msgpack_ext_example_tests.erl` for example code.

```erlang
Packer = fun({ref, Ref}, Opt) when is_reference(Ref) -> {ok, {12, term_to_binary(Ref)}} end,
Unpacker = fun(12, Bin) -> {ok, {ref, binary_to_term(Bin)}} end,
Ref = make_ref(),
Opt = [{ext,{Packer,Unpacker}}],
{ok, {ref, Ref}} = msgpack:unpack(msgpack:pack({ref, Ref}, Opt), Opt).
```

This is still experimental feature, so I'm waiting for your feedback.

## Compatibility mode

To use as same with [old spec](https://github.com/msgpack/msgpack/blob/master/spec-old.md):

```erlang
OldHam = msgpack:pack(Spam, [{enable_str,false}]),
{ok, Spam} = msgpack:unpack(OldHam, [{enable_str,false}]).
```

Since 0.2.3 now it's **false by default**.

## Further tests

See [msgpack-erlang-tests](http://github.com/kuenishi/msgpack-erlang-tests) for further tests

## License

Apache License 2.0

# Release Notes

## 0.4.0

- Deprecate `nil`
- Moved to rebar3
- Promote default map unpacker as default format when OTP is >= 17
- Added QuickCheck tests
- Since this version OTP older than R16B03-1 are no more supported

## 0.3.5 / 0.3.4

- 0.3 series will be the last versions that supports R16B or older
  versions of OTP.
- OTP 18.0 support
- Promote default map unpacker as default format when OTP is >= 18

## 0.3.3

- Add OTP 17 series to Travis-CI tests
- Fix wrong numbering for ext types
- Allow packing maps even when {format,map} is not set
- Fix Dialyzer invalid contract warning
- Proper use of null for jiffy-style encoding/decoding

## 0.3.2

- set back default style as jiffy
- fix bugs around nil/null handling

## 0.3.0

- supports map new in 17.0
- jiffy-style maps will be deprecated in near future
- set default style as map

## 0.2.8

0.2 series works with OTP 17.0, R16, R15, and with MessagePack's new
and old format. But does not support `map` type introduced in
OTP 17.0.

It also supports JSX-compatible mode.
