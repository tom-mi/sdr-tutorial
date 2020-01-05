# SDR Tutorial 1\_00 – Solution

## Analyze the signal

The sample signal has a pulse width of about `20` samples, and shows two gap widths: about `95` and `190` samples.
We just guessed that the a short gap is a `0` and a long gap is a `1`, which works out well as we see in the next section.

## Collect more sample data and reverse-engineer the encoding

From the first two sample files in the `samples` folder, we get the following sample data:

| Temperature | Bit sequence                                      |
| `-5.7 °C`   | `1 1 0 1 0 0 1 0 1 0 0 0 0 1 0 1 0 0 0 0 0 1 1 1` |
| ` 5.1 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 0 1 0 1 0 0 0 0 0 0 0 1` |
| ` 5.8 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 0 1 0 1 0 0 0 0 1 0 0 0` |
| ` 6.5 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 0 1 1 0 0 0 0 0 0 1 0 1` |
| ` 7.0 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0` |
| ` 7.6 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 0 1 1 1 0 0 0 0 0 1 1 0` |
| ` 8.1 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 1` |
| ` 8.6 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0` |
| ` 9.1 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 0 0 1 0 0 0 0 0 0 0 1` |
| `14.2 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 1 0` |
| `14.5 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 1 1 0 0 1 0 0 0 1 0 1` |
| `14.8 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 1 1 0 0 1 0 0 1 0 0 0` |
| `15.4 °C`   | `0 1 0 0 1 0 1 0 0 0 0 0 1 1 1 1 0 0 0 0 0 1 0 0` |
| `16.0 °C`   | `0 1 0 0 1 0 1 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 0` |

By comparing the bit sequences with the temperatures, we observe:
* There is a 8-bit header that is always constant. Might be the channel, or something about the battery status (we'll ignore that for this tutorial).
* The next 8 bits are a signed integer for the degrees in Celsius, where the first bit is a sign bit.
  Note that this is not the usual way to represent signed integers.
* The last 4 bits encode a 10th degree Celsius.
* It is unclear however what the purpose of the 4 bits in between is.


```
 0 1 0 0 1 0 1 0 0 0 0 0 1 0 0 0 0 0 0 0 0 1 1 0 
|               |               |       |       |
| 8-bit header  | 8-bit signed  |  ???  | 0.1°C |
|               | integer °C    |       |       |
```

To decode the bit sequence into a human-readable output, I used the following code in a _Binary message processor_ block:

```
import struct
import numpy

if len(data) == 24:
    byte_data = numpy.packbits(data).tobytes()
    value, fraction = struct.unpack('xbb', byte_data)
    signed_value = value & 0b01111111
    if value & 0b10000000:
        signed_value *= -1
    fraction &= 0b00001111
    return f"Temperature: {signed_value}.{fraction} °C"
```

## Add a message decoder

The complete decoder is here: [decoder.grc](decoder.grc)




# Resources

- [Wikipedia: On-Off Keying](https://en.wikipedia.org/wiki/On%E2%80%93off_keying)
- [Empfang von Wettersensordaten mit RTL-SDR](https://www.kompf.de/weather/rtlsdrsensor.html)
- [GQRX for digital signals](https://www.bastibl.net/gqrx-for-digital-signals/)
