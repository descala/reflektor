Configure an OpenSSH server to act as a reflektor
=================================================

The server running sshd will have a user account for each _user_ and each
_host_. We assume these accounts are already set up.

Reflektor port
--------------

We say a TCP port is a  _reflektor port_ on the server when it is used to
forward sshd connections. The host does a `RemoteForward` of this port
and the user does a `LocalForward` of the same port.

A _reflektor port_ is only bound to localhost, and is firewalled from any
other network interfaces.

Since a host may want to publish many services, the server must allocate a
unique _reflektor port_ for each service.


Host account
------------

The public key of the host should be configured with the following
restrictions:

1. Can only request a RemoteForward on the reflektor ports the server has
   assigned to it
2. Can not do any LocalForward
3. Can not execute any system command

**Example**: _host001_ wants to publish its sshd service. It needs to know its
assigned reflektor port number/s. Lets say it is just port number 12345.

This is the `/home/host001/.ssh/authorized_keys` file to apply the above
restrictions:

```text
PermitRemoteOpen="localhost:12345",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty [host key type] [host public key] 
```

**TODO** The `PermitRemoteOpen` functionality is not implmented in OpenSSH yet.
See [Bug 2038 - permitopen functionality but for remote forwards][1]

With this setup _host001_ connects to server with

```text
ssh -o 'RemoteForward localhost:12345:localhost:22' host001@server
```

This connection must be keept open.

User account
------------

The public key of the host should be configured with the following
restrictions:

1. Can only request a LocalForward to the reflektor ports the user has access
   rights.
2. Can not do any RemoteForward
3. Can not execute any system command


**Example**: _user001_ wants to access sshd running on _host001_. It needs to
know the reflektor port number assigned to "sshd running on host001".

This is the `/home/user001/.ssh/authorized_keys` file to apply the above
restrictions:

```text
PermitOpen="localhost:12345",no-port-forwarding,no-X11-forwarding,no-agent-forwarding,no-pty [user key type] [user public key] 
```
With this setup _user001_ connects to sshd running on _host001_ with

```text
ssh -o 'ProxyCommand ssh -W localhost:12345 user001@server'
```




[1]: https://bugzilla.mindrot.org/show_bug.cgi?id=2038
