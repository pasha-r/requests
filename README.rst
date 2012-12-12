Requests: HTTP for Humans
=========================


.. image:: https://secure.travis-ci.org/kennethreitz/requests.png?branch=develop
        :target: https://secure.travis-ci.org/kennethreitz/requests

Requests is an ISC Licensed HTTP library, written in Python, for human
beings.

Most existing Python modules for sending HTTP requests are extremely
verbose and cumbersome. Python's builtin urllib2 module provides most of
the HTTP capabilities you should need, but the api is thoroughly broken.
It requires an enormous amount of work (even method overrides) to
perform the simplest of tasks.

Things shouldn't be this way. Not in Python.

::

    >>> r = requests.get('https://api.github.com', auth=('user', 'pass'))
    >>> r.status_code
    204
    >>> r.headers['content-type']
    'application/json'
    >>> r.text
    ...

See `the same code, without Requests <https://gist.github.com/973705>`_.

Requests allow you to send HTTP/1.1 requests. You can add headers, form data,
multipart files, and parameters with simple Python dictionaries, and access the
response data in the same way. It's powered by httplib and `urllib3
<https://github.com/shazow/urllib3>`_, but it does all the hard work and crazy
hacks for you.


Features
--------

- International Domains and URLs
- Keep-Alive & Connection Pooling
- Sessions with Cookie Persistence
- Browser-style SSL Verification
- Basic/Digest Authentication
- Elegant Key/Value Cookies
- Automatic Decompression
- Unicode Response Bodies
- Multipart File Uploads
- Connection Timeouts
- Thread-safety


Installation
------------

To install requests, simply: ::

    $ pip install requests

Or, if you absolutely must: ::

    $ easy_install requests

But, you really shouldn't do that.



Contribute
----------

#. Check for open issues or open a fresh issue to start a discussion around a feature idea or a bug. There is a Contributor Friendly tag for issues that should be ideal for people who are not very familiar with the codebase yet.
#. Fork `the repository`_ on Github to start making your changes to the **develop** branch (or branch off of it).
#. Write a test which shows that the bug was fixed or that the feature works as expected.
#. Send a pull request and bug the maintainer until it gets merged and published. :) Make sure to add yourself to AUTHORS_.

.. _`the repository`: http://github.com/kennethreitz/requests
.. _AUTHORS: https://github.com/kennethreitz/requests/blob/develop/AUTHORS.rst



Requests: two timeouts feature
------------------------------

I'd like to suggest a new feature - two types of timeouts. Regular one - happens when we wait for response from server on existing connection. And a new one - connection timeout. Happens only when we try to create a new socket connection. I added new request parameter connect_timeout and two new exceptions ConnectionTimeout and OperationTimeout.
But why anybody would need such a thing?

It can be useful when a call to some HTTP API takes a long time to answer the query (for ex. reading lots of data), but not so much to connect to the server. With this new feature we can set bigger timeout and smaller connect_timeout, so network failures, dead API servers or DNS problems could be detected fast.

Other use case is - write request gone bad. We connected to the server, called PUT to create some new record and caught timeout while waiting for response. Some async APIs work so that your request was actually processed despite broken connection. And with two separate exceptions we can react differently in such a case - on ConnectionTimeout we know for sure nothing've been done so we can repeat the call. On OperationTimeout we may (or may not) poll the server for some indication that something is already going on and if not repeat the call.
On implementation:

I've implemented this feature in a way that does not break expected behavior for an existing code using the library. New exceptions - ConnectionTimeout and OperationTimeout are subclasses of existing Timeout class. Same goes for urllib3 TimeoutError and it's new subclasses.

New parameter connect_timeout is optional and when not given, value of the timeout parameter is used for both timeouts. New HTTPConnectionTwo class (lame name, any suggestions?) takes one timeout in init method and then checks whether it's a single value or a pair, so it also can be initialized exactly like HTTPConnection.
