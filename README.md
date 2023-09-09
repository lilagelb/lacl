# LACL

lilagelb's Application Configuration Language, LACL for short, is a configuration markup language designed for the configuration of application settings and preferences. It is *not* designed to be a serialisation format for complex data like XML or YAML. It is meant to be written, read, and edited by a human and interpreted by a machine, *not* to be written by a machine.

It was designed to be simple and powerful; in particular, it is deliberately designed not to have more than one way of producing the same data structure, a feature found in pretty much every other language used for configuration (LACL has no dictionary-like structure for this reason). This makes LACL files quick and easy to write - there's one way to represent it, end of.

It was also designed to minimise boilerplate and maximise flexibility, born out of a frustration at configuring various applications and utilities in which there was no way, for example, to say 'the style of a mutable variable should be the style of a standard variable but emboldened'.

The specification can be found [here](specification.md).
