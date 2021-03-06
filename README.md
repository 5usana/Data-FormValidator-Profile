# NAME

Data::FormValidator::Profile - Profile object for Data::FormValidator

# SYNOPSIS

```perl
use Data::FormValidator;
use Data::FormValidator::Profile;

# create a new DFV::Profile object
my $profile = Data::FormValidator::Profile->new( {
    optional  => [qw( this that )],
    required  => [qw( some other thing )],
    } );

# query the optional/required fields in the profile
my @optional = $profile->optional();
my @required = $profile->required();

# reduce the profile to just a limited set of fields
$profile->only( qw(this that) );

# remove fields from the profile
$profile->remove( qw(some other thing) );

# add a new field to the profile
$profile->add( 'username',
    required    => 1,
    filters     => 'trim',
    constraints => FV_max_length(32),
    msgs => {
        constraints => {
            max_length => 'too big',
        },
    },
);

# call chaining, to make manipulation quicker
$profile->only(qw( this that other ))
  ->remove(qw( that ))
  ->add(qw( foo ))
  ->check({ this => 'is a test' });

# use the profile to validate data
my $res = $profile->check({ this => 'is a test' });
# ... or
$res = Data::FormValidator->check(
  { this => 'is a test' },
  $profile->profile(),
);
```

# DESCRIPTION

`Data::FormValidator::Profile` provides an interface to help manage
`Data::FormValidator` profiles.

I found that I was frequently using `Data::FormValidator` profiles to help
define my DB constraints and validation rules, but that depending on the
context I was working in I may only be manipulating a small handful of the
fields at any given point.  Although I could query my DB layer to get the
default validation profile, I was really only concerned with the rules for two
or three fields.  Thus, `Data::FormValidator::Profile`, to help make it easier
to trim profiles to include only certain sets of fields in the profile.

## Limitations

All said, though, `Data::FormValidator::Profile` has some limitations that you
need to be aware of.

- It **only** removes fields from the following profile attributes:

    ```
    required
    optional
    defaults
    field_filters
    constraints
    constraint_methods
    ```

    **NO** effort is made to update dependencies, groups, require\_some, or anything
    based on a regexp match.  Yes, that does mean that this module is limited in
    its usefulness if you've got really fancy `Data::FormValidator` profiles.
    That said, though, I'm not using anything that fancy, so it works for me.

- To use the profile with `Data::FormValidator`, use either the form of:

    ```
    $profile->check($data)
    ```

    or

    ```
    Data::FormValidator->check($data, $profile->profile)
    ```

    `Data::FormValidator` won't accept a blessed object when calling
    `Data::FormValidator->check()`, so you need to call
    `$profile->profile()` to turn the
    profile into a HASHREF first.

    Unless you're doing anything fancier and you've got an actual
    `Data::FormValidator` object that you're working with, its easier/simpler to
    just call `$profile->check($data)`; that's the recommended interface.

# METHODS

- new()

    Creates a new DFV::Profile object, based on the given profile (which can be
    provided either as a HASH or a HASHREF).

- check($data)

    Checks the given `$data` against the profile. This method simply acts as a
    short-hand to
    `Data::FormValidator->check($data,$profile->profile)`.

- profile()

    Returns the actual profile, as a hash-ref. You need to call this method
    when you want to send the profile through to `Data::FormValidator` to do
    data validation.

- required()

    Returns the list of "required" fields in the validation profile.

- optional()

    Returns the list of "optional" fields in the validation profile.

- only(@fields)

    Reduces the profile so that it only contains information on the given list
    of `@fields`.

    Returns `$self`, to support call-chaining.

- remove(@fields)

    Removes any of the given `@fields` from the profile.

    Returns `$self`, to support call-chaining.

- make\_optional(@fields)

    Ensures that the given set of `@fields` are set as being optional (even if
    they were previously described as being required fields).

    Returns `$self`, to support call-chaining.

- make\_required(@fields)

    Ensures that the given set of `@fields` are set as being required (even if
    they were previously described as being optional fields).

    Returns `$self`, to support call-chaining.

- set(%options)

    Explicitly sets one or more `%options` into the profile. Useful when you
    KNOW exactly what you want to add/do to the profile.

    Returns `$self`, to support call-chaining.

- add($field, %args)

    Adds the given `$field` to the validation profile, and sets up additional
    validation rules as per the provided `%args`.

    If the field already exists in the profile, this method throws a fatal
    exception.

    Returns `$self`, to support call-chaining.

    Acceptable `%args` include:

    - required

        If non-zero, specifies that the field is required and is not an optional
        field (default is to be optional)

    - default

        Default value for the field.

    - dependencies

        "dependencies" for this field. Replaces existing value.

    - filters

        "field\_filters" to be applied. Replaces existing value.

    - constraints

        "constraint\_methods" for this field. Replaces existing value.

    - msgs

        Hash-ref of "constraint messages" that are related to this field. Replaces
        existing values.

    Here's an example to help show how the `%args` are mapped into a
    validation profile:

    ```perl
    $profile->add(
        'username',
        required    => 1,
        filters     => ['trim', 'lc'],
        constraints => FV_length_between(4,32),
        msgs => {
            length_between => 'Username must be 4-32 chars in length.',
            },
        );
    ```

    becomes:

    ```perl
    {
        required => [qw( username )],
        field_filters => {
            username => ['trim', 'lc'],
        },
        constraint_methods => {
            username => FV_length_between(4,32),
        },
        msgs => {
            constraints => {
                length_between => 'Username must be ...',
            },
        },
    }
    ```

# AUTHOR

Graham TerMarsch (cpan@howlingfrog.com)

# COPYRIGHT

Copyright (C) 2008, Graham TerMarsch.  All Rights Reserved.

This is free software; you can redistribute it and/or modify it under the same
terms as Perl itself.

# SEE ALSO

[Data::FormValidator](https://metacpan.org/pod/Data%3A%3AFormValidator).
