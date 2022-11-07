# stun-bytes

A low-level base for STUN message protocol parsers.

## STUN Message structure

### Header
```text
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|    Message Type (16 bits)   |     Message Length (16 bits)    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                     Magic Cookie (32 bits)                    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                    Transaction ID (96 bits)                   |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

**Note:** since [RFC5389](https://www.rfc-editor.org/rfc/rfc5389#section-6),
`Transaction ID` field has been split into `Magic Cookie` and `Transaction ID`.
If you want to follow this split, enable the `cookie` feature.

### Attribute
```text
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|        Type (16 bits)         |        Length (16 bits)       |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                         Value (variable)                     ..
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Examples

### Parse STUN message

```rust
const MSG: [u8; 28] = [
    0x00, 0x01,                     // type: Binding Request
    0x00, 0x08,                     // length: 8 (header does not count)
    0x21, 0x12, 0xA4, 0x42,         // magic cookie
    0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x01,         // transaction id
    0x00, 0x03,                     // type: ChangeRequest
    0x00, 0x04,                     // length: 4 (only value bytes count)
    0x00, 0x00, 0x00, 0x40 | 0x20,  // change both ip and port
];

let msg = ByteMsg::from_arr(&MSG);

assert_eq!(&MSG[0..2], msg.typ().unwrap());     // read type field
assert_eq!(&MSG[2..4], msg.len().unwrap());     // read length field
assert_eq!(&MSG[4..8], msg.cookie().unwrap());  // read cookie field (enable 'cookie' feature first)
assert_eq!(&MSG[8..20], msg.tid().unwrap());    // read transaction id field
assert_eq!(&MSG[20..28], msg.attrs().unwrap()); // read all attribute bytes

let attr = msg.attrs_iter().next().unwrap();    // iterate over attributes

assert_eq!(&MSG[20..22], attr.typ().unwrap());  // read attribute type field
assert_eq!(&MSG[22..24], attr.len().unwrap());  // read attribute length field
assert_eq!(&MSG[24..28], attr.val().unwrap());  // read attribute value field
```

### Create STUN message

```rust
const MSG: [u8; 28] = [
    0x00, 0x01,                     // type: Binding Request
    0x00, 0x08,                     // length: 8 (header does not count)
    0x21, 0x12, 0xA4, 0x42,         // magic cookie
    0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x00,
    0x00, 0x00, 0x00, 0x01,         // transaction id
    0x00, 0x03,                     // type: ChangeRequest
    0x00, 0x04,                     // length: 4 (only value bytes count)
    0x00, 0x00, 0x00, 0x40 | 0x20,  // change both ip and port
];

let mut buf = [0u8; MSG.len()];
let mut msg = ByteMsgMut::from_arr_mut(&mut buf);

msg.typ().unwrap().copy_from(MSG.carved().unwrap());                                    // write type field
// msg.len().unwrap().copy_from(MSG.carve(2).unwrap());                                 // length field updates automatically
msg.cookie().unwrap().copy_from(MSG.carve(4).unwrap());                                 // write cookie field
msg.tid().unwrap().copy_from(MSG.carve(8).unwrap());                                    // write transaction id field
msg.add_attr(MSG.carve(20).unwrap(), MSG.carve(22).unwrap(), MSG.get(24..28).unwrap()); // write attribute (type, length, value)

assert_eq!(&MSG, msg.as_bytes());
```

## Principles
 - Zero-copy


 - No-std


 - RFC-agnostic

## Contribution guidelines

Pull requests are welcome. Please make sure your contribution adheres to the [Principles](#Principles) section above.
