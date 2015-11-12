---
layout: post
title: Python socket - Y U No close()?
categories: python
---

So I'm writing some python that does IPC over a local socketpair, and
it hangs during a clean connection teardown.  The code in question
calls `close()` on the socket's file object and then waits for another
"reader" thread that is blocked reading from the socket to notice the
close and exit.  Nothing very interesting.

Except when I `strace` the process, *there is no close() syscall
happening!* After much head scratching and convincing myself that my
code is doing what I think it is doing, I go read the (python2.7) code
in question:

{% highlight python %}
# /usr/lib/python2.7/socket.py
class _fileobject(object):
    """Faux file object attached to a socket object."""

    def __init__(self, sock, mode='rb', bufsize=-1, close=False):
        # ... removed for clarity ...
        self._close = close

    def close(self):
        try:
            if self._sock:
                self.flush()
        finally:
            if self._close:
                self._sock.close()
            self._sock = None
{% endhighlight %}

.. so what this is saying is that by default `_close=False` and
`self._sock.close()` is **never called**.  Oh, ok, I can sort of see
why that might be useful in odd circumstances, I'll just pass
`close=True` to `makefile()`:

{% highlight python %}
class _socketobject(object):

    def makefile(self, mode='r', bufsize=-1):
        return _fileobject(self._sock, mode, bufsize)
{% endhighlight %}

Wait, what?  That's right, there's no way to actually pass
`close=True`.  Huh.

Oh well, I already had this other change to refactor this code to use
sockets directly rather than file-like objects, so I'll just use
that.  *\<insert typing noise...\>*  Ok, let's go!

Same hang-on-clean-shutdown problem.

Huh.  `strace` again, *still* no `close()` syscall!  Sure enough, in the
(python2.7) socket class:

{% highlight python %}
class _socketobject(object):

    def close(self, _closedsocket=_closedsocket,
              _delegate_methods=_delegate_methods, setattr=setattr):
        # This function should not reference any globals. See issue #808164.
        self._sock = _closedsocket()
        dummy = self._sock._dummy
        for method in _delegate_methods:
            setattr(self, method, dummy)
{% endhighlight %}

... wait, what?  No matter how many times I re-read this code, I still
can't find an actual `close()` call in there.  I gather this code is
replacing the `_sock` object with a dummy `_closedsocket()` object and
then just waiting for garbage collection to actually close the real
socket.  This just isn't good enough for my case, since I'm about to
go and do a blocking operation that (naively!) assumed `close()` meant
`close()` and garbage collection may *never* happen!  I presume it was
done this way because writing code that works during global
destruction is difficult since you can't count on anything actually
existing anymore - but seriously.

*TL;DR:* python is bonkers, and you should just use
`socket.shutdown(socket.SHUT_RDWR)` instead because the python people
haven't got to it and broken that one yet.

More to the point, why is *any* of this here?  If we need the socket
to behave differently, why not just fix the actual (cpython) socket
class?  `socket.py` starts with:

{% highlight python %}
from _socket import socket
_realsocket = socket

# ...

socket = SocketType = _socketobject
{% endhighlight %}

If you have to monkey-patch the only user of your class, in the same
software bundle in which you ship both classes, then you're probably
doing it wrong.

This concludes today's python frustration.
