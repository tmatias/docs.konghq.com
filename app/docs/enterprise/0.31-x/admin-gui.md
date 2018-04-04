---
title: Kong Admin GUI
class: page-install-method
---

# Kong Admin GUI

## Access the GUI with SSL enabled
1. **Configure**
  
    The Admin GUI configuration settings in `kong.conf` to enable SSL.

> Note: starting from Kong 0.32, Admin GUI listen directives follow [the new format](https://getkong.org/docs/0.13.x/configuration/#admin_listen) for Kong listen directives. Make sure `admin_gui_listen` is adapted accordingly.

2. **Navigate**
  
    Go to your configured Admin GUI `ip:port` using the https:// protocol.

3. **Confirm** 
  
    If you are using Kong 0.30 or newer, make sure that the `admin_listen` configuration value binds to the desired network interface. By default, and for security reasons, this setting binds to the local interface only.

> Note: If you access the Admin GUI over the `http://` protocol the Admin GUI will not attempt to connect to the Kong Admin API over SSL.
