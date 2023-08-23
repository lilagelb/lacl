# LACL

lilagelb's Application Configuration Language, LACL for short, is a configuration markup language designed for the configuration of application settings and preferences. It is *not* designed to be a serialisation format for complex data like XML or YAML. It is meant to be written, read, and edited by a human and interpreted by a machine, *not* to be written by a machine.

It was designed to be simple and powerful; in particular, it is deliberately designed not to have more than one way of producing the same data structure, a feature found in pretty much every other language used for configuration (LACL has no dictionary-like structure for this reason). This makes LACL files quick and easy to write - there's one way to represent it, end of.

The specification is [here](specification.md).
