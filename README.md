# NAME

Data::MethodProxy - Integrate dynamic logic with static configuration.

# SYNOPSIS

    use Data::MethodProxy;
    
    $config = get_your_config_somewhere();
    $config = apply_method_proxies( $config );

# DESCRIPTION

A method proxy is a particular data structure which, when found,
is replaced by the value returned by calling that method.  In this
way static configuration can be setup to call your code and return
dynamic contents.  This makes static configuration much more powerful,
and gives you the ability to be more declarative in how dynamic values
make it into your configuration.

# EXAMPLE

Consider this static YAML configuration:

    ---
    db:
        dsn: DBI:mysql:database=foo
        username: bar
        password: abc123

Putting your database password inside of a configuration file is usually
considered a bad practice.  You can use a method proxy to get around this
without jumping through a bunch of hoops:

    ---
    db:
        dsn: DBI:mysql:database=foo
        username: bar
        password:
            - $proxy
            - MyApp::Config
            - get_db_password
            - bar

When ["apply\_method\_proxies"](#apply_method_proxies) is called on the above data structure it will
see the method proxy and will replace the array ref with the return value of
calling the method.

A method proxy, in Perl syntax, looks like this:

    ['$proxy', $package, $method, @args]

The `$proxy` string can also be written as `&proxy`.  The above is then
converted to a method call and replaced by the return value of the method call:

    $package->$method( @args );

In the above database password example the method call would be this:

    MyApp::Config->get_db_password( 'bar' );

You would still need to create a `MyApp::Config` package, and add a
`get_db_password` method to it.

# FUNCTIONS

Only the ["apply\_method\_proxies"](#apply_method_proxies) function is exported by default.

## apply\_method\_proxies

    $config = apply_method_proxies( $config );

Traverses the supplied data looking for method proxies, calling them, and
replacing them with the return value of the method.  Any value may be passed,
such as a hash ref, an array ref, a method proxy, an object, a scalar, etc.
Array and hash refs will be recursively searched for method proxies.

If a circular reference is detected an error will be thrown.

## is\_method\_proxy

    if (is_method_proxy( $some_data )) { ... }

Returns true if the supplied data is an array ref where the first value
is the string `$proxy` or `&proxy`.

## call\_method\_proxy

    call_method_proxy( ['$proxy', $package, $method, @args] );

Calls a method proxy and returns the value.

# AUTHOR

Aran Clary Deltac <bluefeet@gmail.com>

# ACKNOWLEDGEMENTS

Thanks to [ZipRecruiter](https://www.ziprecruiter.com/)
for encouraging their employees to contribute back to the open
source ecosystem.  Without their dedication to quality software
development this distribution would not exist.

# LICENSE

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.
