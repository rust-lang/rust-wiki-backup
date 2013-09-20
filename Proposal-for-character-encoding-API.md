* [Introduction and initial proposal](https://mail.mozilla.org/pipermail/rust-dev/2013-September/005503.html)
* [WHATWG Encoding spec](http://encoding.spec.whatwg.org/)

## Current proposal

```rust
/// Each implementation of Encoding has one corresponding implementation
/// of Decoder (and one of Encoder).
trait Decoder {
     /// Simple, "one shot" API.
     /// Decode a single byte string that is entirely in memory.
     /// May raise the decoding_error condition.
     fn decode(input: &[u8]) -> Result<~str, DecodeError> {
         // Implementation left out.
         // This is a default method, but not meant to be overridden.
     }

     /// A new Decoder instance should be used for every input.
     /// A Decoder instance should be discarded after finish() was called
     /// or DecodeError was returned.
     fn new() -> Self;

     /// Call this repeatedly with a chunck of input bytes.
     /// As much as possible of the decoded text is appended to output.
     /// May raise the decoding_error condition.
     fn feed<W: StringWriter>(&self, input: &[u8], output: &mut W)
                           -> Option<DecodeError>;

     /// Call this to indicate the end of the input.
     /// The Decoder instance should be discarded afterwards.
     /// Some encodings may append some final output at this point.
     /// May raise the decoding_error condition.
     fn finish<W: StringWriter>(&self, output: &mut W)
                             -> Option<DecodeError>;
}

/// Takes the invalid byte sequence.
/// Return a replacement string, or None to abort with a DecodeError.
condition! {
     pub decoding_error : ~[u8] -> DecodingErrorResult;
}

enum DecodingErrorResult {
     AbortDecoding,
     ReplacementString(~str),
     ReplacementChar(char),
}

/// Functions to be used with decoding_error::cond.trap
mod decoding_error_handlers {
     fn abort(_: ~[u8]) -> DecodingErrorResult { AbortDecoding }
     fn replacement(_: ~[u8]) -> DecodingErrorResult { ReplacementChar('\uFFFD') }
}

struct DecodeError {
     input_byte_offset: uint,
     invalid_byte_sequence: ~[u8],
}

trait StringWriter {
     fn write_char(&mut self, c: char);
     fn write_str(&mut self, s: &str);
}


/// Only supports the set of labels defined in the spec
/// http://encoding.spec.whatwg.org/#encodings
/// Such a label can come eg. from an HTTP header:
/// Content-Type: text/plain; charset=<label>
fn encoding_from_label(label: &str) -> &'static Encoding {
     // Implementation left out
}

/// Types implementing this trait are "algorithms"
/// such as UTF8, UTF-16, SingleByteEncoding, etc.
/// Values of these types are "encodings" as defined in the WHATWG spec:
/// UTF-8, UTF-16-LE, Windows-1252, etc.
trait Encoding {
     /// Could become an associated type with a ::new() constructor
     /// when the language supports that.
     fn new_decoder(&self) -> ~Decoder;

     /// Simple, "one shot" API.
     /// Decode a single byte string that is entirely in memory.
     /// May raise the decoding_error condition.
     fn decode(&self, input: &[u8]) -> Result<~str, DecodeError> {
         // Implementation (using a Decoder) left out.
         // This is a default method, but not meant to be overridden.
     }
}
```