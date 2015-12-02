---
layout: post
title: Inhibiting Suspend
---

When I'm away from my home network, I sometimes need to ssh in
remotely in order to fetch that file or continue working on that
project that I haven't published publicly yet.  The complication is
that I configure my desktop to suspend itself when it is idle to save
power.  I use wake-on-lan to wake up my desktop, which is easy enough,
but the screensaver is never unlocked and so after a few minutes the
fancy desktop machinery triggers a suspend again!

If only there was some way to let the power management system know
about my ssh session...

It turns out exactly this functionality is exposed via D-Bus.  I wrote
the following `pminhibit` script to send the correct
`/org/freedesktop/PowerManagement/Inhibit` message, and then suspend
will be prevented *so long as this process is still running*:

{% highlight python %}
#!/usr/bin/python

import dbus
import os
import signal
import sys


VERBOSE = True


class Inhibit(object):
    def __init__(self):
        self.cookie = None
        self.pm = dbus.SessionBus().get_object(
            'org.freedesktop.PowerManagement',
            '/org/freedesktop/PowerManagement/Inhibit')
        if VERBOSE and self.pm.HasInhibit():
            print 'Btw, power management was already inhibited'

    def __enter__(self):
        self.cookie = self.pm.Inhibit(sys.argv[0], 'User inhibiting suspend')
        assert self.pm.HasInhibit()

    def __exit__(self, exc_type, exc_value, traceback):
        self.pm.UnInhibit(self.cookie)
        if VERBOSE and self.pm.HasInhibit():
            print 'Btw, power management is still inhibited'
        return False


def main():
    if not os.environ.get('DISPLAY', ''):
        os.putenv('DISPLAY', ':0')

    def sighandler(signum, frame):
        sys.exit(0)
    for s in (signal.SIGHUP, signal.SIGTERM, signal.SIGINT):
        signal.signal(s, sighandler)

    with Inhibit():
        while True:
            signal.pause()


if __name__ == '__main__':
    main()
{% endhighlight %}

I then add the following to `~/.profile` but you will need to check
that your shell kills the subprocess correctly when you log out (I use
zsh, but I believe bash is more eager to disown backgrounded processes):

{% highlight shell %}
# If this is an interactive ssh session, inhibit suspend
if [ -t 0 -a -n "$SSH_CONNECTION" ]; then
    pminhibit &
fi
{% endhighlight %}

It works a treat!
