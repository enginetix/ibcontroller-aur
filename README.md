IBController AUR
=================
Source and documentation for the
[Interactive Brokers](http://interactivebrokers.com/)
[IBController](https://github.com/ib-controller/ib-controller)
AUR package [ib-controller](https://aur.archlinux.org/packages/ib-controller/).

This package depends on
[Trader Workstation](http://www.interactivebrokers.com/en/pagemap/pagemap_APISolutions.php)
having been installed via AUR package
[ib-tws](https://aur.archlinux.org/packages/ib-tws/). It installs a custom build
of IBController and a headless 
[systemd](http://en.wikipedia.org/wiki/Systemd) configuration that supports
multiple concurrent IB Gateway instances while addressing usual security needs.

To start IBController and the IB Gateway it manages, create an INI file in
``/etc/ibcontroller`` and use systemd commands such as:

```
sudo systemctl start ibcontroller@ininame.service
sudo systemctl enable ibcontroller@ininame.service
```

The aforementioned ``ininame`` should be the simple name of an ``/etc/ibcontroller``
INI file. For example, use ``ibcontroller@fdemo.service`` for the included
[financial advisor sample INI file](package/fdemo.ini),
or ``ibcontroller@edemo.service`` for the included
[individual user sample INI file](package/edemo.ini). Both sample INI files can
be used concurrently, as they bind to unique ports. Please refer to the
[IBController](https://github.com/ib-controller/ib-controller) documentation for
the meaning of individual INI configuration settings.

Please note future updates to this package may change the sample INI files to
reflect new configuration defaults. You should create your own
INI file(s) rather than editing one of the shipped INI files. Similarly the
systemd configuration file may be changed in future updates.

Limitations
-----------
Systemd recommends
[sd_notify](http://www.freedesktop.org/software/systemd/man/sd_notify.html)
calls for its watchdog process monitoring and to determine when a process has
fully loaded.

IBController and IB Gateway are written in Java. I was unable to locate an
``sd_notify`` library for the JVM at this time. If one becomes available, please
contribute this improvement to IBController so there is more comprehensive
systemd integration. In the interim you should have your trading systems detect
the unavailability of IB Gateway and report it via an application-specific
mechanism. Please also remember IB performs a daily restart of its servers, so
you should always expect some service disconnections.

Security
--------
Always ensure the ``/etc/ibcontroller`` INI files are only readable by ``root``,
as they contain your IB credentials. If you require additional credential safety
you may like to consider the IBController ``PasswordEncrypted`` option, however
it is easily decrypted and therefore not used in the sample files.

There is no mechanism to use IB hardware security tokens with the systemd
configuration. This is due to the systemd configuration using a virtual
framebuffer, so there is no mechanism by which hardware challenges can be
presented. If you have a hardware token, you can disable it for trading system
access via IB Account Management. Alternately you may like to create a new user
account under your IB account which has trading access but no hardware token.

Only 127.0.0.1 is trusted by the resulting IB Gateway instance. Port forwarding
(eg iptables, SSH tunneling) is suggested if other IP addresses are required.

Build and Test
--------------
If you'd like to try out changes to the package, these commands offer a start:

````
cd package
rm -rf pkg src && makepkg -f
namcap -m *.xz
sudo pacman -U *.xz
sudo systemctl daemon-reload
sudo systemctl start ibcontroller@fdemo.service
sudo systemctl status ibcontroller@fdemo.service
sudo systemctl stop ibcontroller@fdemo.service
rm -f *.gz && makepkg --source
burp -c daemons *.gz
````