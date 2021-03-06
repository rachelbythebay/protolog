# protolog, or binary logging for Apache using protocol buffers.

If you're reading this, odds are good that you are looking for a way to 
log hits from your Apache web server without generating ASCII spew 
which is hard to parse later.  If so, you are in the right place.  Here 
are some notes about making it work.


## Limitations

This software definitely does not attempt to solve every problem.  If it 
works for you, great!  If not, prepare to get your hands dirty.


### Log path

Use the ProtoLog directive in your config file to tell this module
where to log.  If you install this module and nothing happens, odds
are you forgot to add this directive somewhere!

    ProtoLog /var/log/httpd/proto.log

This will work both at the global level and in VirtualHost containers.
If a VirtualHost does not contain a ProtoLog entry, it will use the
global log(s).

## Dependencies

### protobuf-c

You'll need protobuf-c installed to make this work.  Some distributions 
package this and others do not.  It's not hard to build from source, 
though.  Once "protoc-c" is in your path, it should be usable to build 
protolog.


### Apache development stuff

You also need a working installation of Apache with the dev stuff 
included.  This may mean installing extra packages on your system.  If 
you have "apxs" in your path, you're probably set.


### Apache versions

I wrote this against Apache 2.2, but it might work on other versions.  
Your mileage may vary.


### Mac builds and protobuf-c

If you really want to run this on a Mac install of Apache for some 
reason, use Macports to install protobuf-c first, then add 
"-I/opt/local/include -L/opt/local/lib" to the build command for 
protolog.so, like this:

  $(APXS) -lprotobuf-c -I/opt/local/include -L/opt/local/lib -c $^

If Macports is not your thing, you'll have to find and add the 
appropriate paths for the includes and libraries, respectively.
I don't feel like torturing myself with autoconf today.

## Build

"make" should do the right thing.  There isn't much going on here.

## Installation

This is really simple.  Drop the protolog.so found in .libs into some 
path like /etc/httpd/modules, then add this to your Apache config 
somewhere:

    LoadModule protolog_module modules/protolog.so

Adjust the path to suit your system, naturally.  Personally, I have that 
one line in /etc/httpd/conf.d/protolog.conf to keep it separate from 
other things.

After that, restart your server and check the log file.  It should pop 
into existence and then start growing automatically.

## Encoding

The log file will contain a series of "netstrings", each containing a 
protocol buffer message which has been serialized to a stream of bytes.

A netstring is just this:

  <length>:<data>,

Or, in other words,

  5:hello,

That's it.  Dig around on Wikipedia or cr.yp.to if you want to know 
more about this format, but it's really just that simple.

More on protocol buffers can be found at the following URL:
http://code.google.com/p/protobuf.


## Fields

The fields in apachelog.proto are primarily the things I found 
interesting while first writing this.  It's trivial to add more, and 
it's also trivial to not use any of them due to the wonder of 
"optional".

The only thing you really do not want to do is change the number of a 
field after it has been used.  If you do that, you will unleash the 
demons of uncertainty upon your logs.  Don't do this!

Obviously, the potential for much badness exists here if people fork 
this thing like crazy, add a bunch of overlapping fields, and then 
expect compatibility down the road.  All I can say about that is to 
repeat myself from before: don't do it.  Work together to coordinate 
these things, and analysis tool compatibility will be your reward.

## Decoding

Assuming you have a simple state machine to extract the data from inside 
each netstring, then you can just pass the <data> directly to protobuf.

In C++, it looks like this:

  LogEntry le;
  if (le.ParseFromString(extracted_data_from_netstring)) {
    DoSomething();
  }

Java and Python will be slightly different but the same general 
principle applies.

## Contact

Need to send me a message?  Try this first:

  http://rachelbythebay.com/contact/

It's my attempt at keeping my e-mail addresses from being harvested.
I'll reply via normal e-mail if you give me an address in your comment.
