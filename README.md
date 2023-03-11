# NAME

DateTimeX::Create - Extend DateTime by adding a convenient create() method.

# SYNOPSIS

        use DateTimeX::Create; # adds create() method to DateTime
        my $dt1 = DateTime->create(2023, 03, 01, 0, 0, 0, 'America/Chicago');
        my $dt2 = DateTime->create(time);                  # time since epoch
        my $dt3 = DateTime->create('1978-07-04 20:18:45'); # parses ISO-like string

# DESCRIPTION

This module offers a create() class method that can be exported into DateTime
or another specified module. It may also be used without exporting anything. It
returns new DateTime objects.

The motivation behind this module is the verbosity of creating DateTime objects:

        my $datetime = DateTime->new(
                year => 2000,
                month => 1,
                day => 1,
                hour => 0,
                minute => 0,
                second => 0
        );

That is a lot of typing just to create one datetime object. Since the DateTime
module does not parse date strings, users are instead directed to CPAN to
choose from a bewildering array of other modules to do this for them.

This module takes a "do what I mean" approach and attempts to parse datetimes
passed as either a list, arrayref, an epoch time, or an ISO-style string.

The most simple use is to call DateTime->create with no arguments which returns
a DateTime object equivalent to 0000-01-01 00:00:00.

# EXPORTING

By default this module exports the create() method to the DateTime package.
You can specify that this module exports its create method to a different
namspace instead of to DateTime (or not to export anything) by passing
arguments to its import method via use:

        use DateTimeX::Create (export_to => 'My::DateTime::Class');
        My::DateTime::Class->create(...); # returns Dat

        use DateTimeX::Create (); # no exports
        DateTimeX::Create->create(...); # returns DateTime objects
        DateTimeX::Create::create('My::DateTime', ...) # returns My::DateTime objects

Exporting to multiple different namespaces is best done by calling import
directly:

        require DateTimeX::Create;
        DateTimeX->import(export_to => 'My::DateTime::Class');
        DateTimeX->import(export_to => 'My::Other::DateTime::Class');

Note that this module does NOT export anything to the caller's namespace. The
following forms will NOT export create() to My::Caller:

        package My::Caller {
                use DateTimeX::Create;
                use DateTimeX::Create qw(create);
        }
        My::Caller->create(...) # error

# PUBLIC METHODS

There is only one method intended for public consumption, which is the
create() class method. It can be used with three different kinds of arguments.

- create(list) OR create(arrayref)

    A list is interpreted as containing elements necessary to create a DateTime in 
    descending order, in other words, year, month, day, hour, minute, second,
    and optional nanosecond, in that order.

    Optionally, a time\_zone name may be supplied at the beginning or end of the list.
    This string will be checked using the DateTime::TimeZone->is\_valid\_name method.

            $dt = DateTime->create(2020, 01, 01, 8, 30, 0, 'America/Chicago');
            # 2020-01-01T08:30:00

    The default year is 0. The default month and day are 1. The default hour,
    minute, and second are 0.

    In create(list) form, either zero or more than one element is required (as
    a single element would be interpreted as a string or seconds since epoch), but
    any element may be undef.

    In create(arrayref) form, all elements may be undef or
    missing.

            $dt = DateTime->create;               # 0000-01-01T00:00:00
            $dt = DateTime->create([]);           # same
            $dt = DateTime->create(undef, undef); # same


            $dt = DateTime->create(2020,  undef); # 2020-01-01T00:00:00

    Be careful not to supply just a year in list form, as this is interpreted as
    an epoch time:

            $dt = DateTime->create(2020); # 1970-01-01T00:33:40

- create(number)

    An integer or float is interpreted as an epoch time and is passed directly
    to DateTime's from\_epoch() class method.

- create(string)

    If the argument to create() is neither a list nor a number, it is assumed to
    be an ISO8601 style date string.

    This module will try to parse strings it thinks are ISO 8601 datetimes using
    its own internal parser (regex). It will match any forms like the following
    (the separator between the date and time may be a space or the letter T):

            YYYY-MM-DD HH:MM:SS
            YYYY-MM-DD HH:MM:SS.ssssss
            YYYY-MM-DD HH:MM:SSZ
            YYYY-MM-DD HH:MM:SS[+-]NN
            YYYY-MM-DD HH:MM:SS[+-]NN:NN

    The internal parser cannot handle unusual cases, such as dates before year 0000
    or after year 9999, for example.

    If unsuccessful, the module DateTime::Format::ISO8601 is used (if available)
    to attempt to parse it instead.

# DEBUGGING

The following package globals may assist in debugging.

- $DateTimeX::Create::looks\_like

    What this module thinks the most recent argument type was. Contains one of the
    strings 'empty', 'list', 'arrayref', 'epoch', or 'iso\_string'. May be undef.

- $DateTimeX::Create::parser\_used

    If used to parse an ISO-style string, may be 'internal' or 'external', with
    'external' referring to DateTime::Format::ISO8601;

- $DateTimeX::Create::force\_parser

    If this variable equals the string 'internal' or 'external', the create() method
    will only attempt to use only the corresponding parser (as described above). The
    default is undef, which will favor the internal parser.

# DEPENDENCIES

- perl v5.36 or greater

    This module uses Perl's new native subroutine signatures. While this module is
    a simple one that could have easily been written for a (much) older version, of
    perl, I believe it is in the community's interest to encourage upgrading.

- Regexp::Common
- DateTime
- Try::Tiny
- Test::More for the test suite

# AUTHOR

Dondi Michael Stroma, <dstroma@gmail.com>

# COPYRIGHT AND LICENSE

Copyright (C) 2023 by Dondi Michael Stroma

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
