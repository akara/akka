.. _io-java-udp:

Using UDP
=========

.. warning::

  The IO implementation is marked as **“experimental”** as of its introduction
  in Akka 2.2.0. We will continue to improve this API based on our users’
  feedback, which implies that while we try to keep incompatible changes to a
  minimum the binary compatibility guarantee for maintenance releases does not
  apply to the contents of the `akka.io` package.

UDP is a connectionless datagram protocol which offers two different ways of
communication on the JDK level:

 * sockets which are free to send datagrams to any destination and receive
   datagrams from any origin

 * sockets which are restricted to communication with one specific remote
   socket address

In the low-level API the distinction is made—confusingly—by whether or not
:meth:`connect` has been called on the socket (even when connect has been
called the protocol is still connectionless). These two forms of UDP usage are
offered using distinct IO extensions described below.

Unconnected UDP
---------------

Simple Send
^^^^^^^^^^^

.. includecode:: code/docs/io/UdpDocTest.java#sender

The simplest form of UDP usage is to just send datagrams without the need of
getting a reply. To this end a “simple sender” facility is provided as
demonstrated above. The UDP extension is queried using the
:meth:`simpleSender` message, which is answered by a :class:`SimpleSenderReady`
notification. The sender of this message is the newly created sender actor
which from this point onward can be used to send datagrams to arbitrary
destinations; in this example it will just send any UTF-8 encoded
:class:`String` it receives to a predefined remote address.

.. note::

  The simple sender will not shut itself down because it cannot know when you
  are done with it. You will need to send it a :class:`PoisonPill` when you
  want to close the ephemeral port the sender is bound to.

Bind (and Send)
^^^^^^^^^^^^^^^

.. includecode:: code/docs/io/UdpDocTest.java#listener

If you want to implement a UDP server which listens on a socket for incoming
datagrams then you need to use the :meth:`bind` command as shown above. The
local address specified may have a zero port in which case the operating system
will automatically choose a free port and assign it to the new socket. Which
port was actually bound can be found out by inspecting the :class:`Bound`
message.

The sender of the :class:`Bound` message is the actor which manages the new
socket. Sending datagrams is achieved by using the :meth:`send` message type
and the socket can be closed by sending a :meth:`unbind` command, in which
case the socket actor will reply with a :class:`Unbound` notification.

Received datagrams are sent to the actor designated in the :meth:`bind`
message, whereas the :class:`Bound` message will be sent to the sender of the
:meth:`bind`.

Connected UDP
-------------

The service provided by the connection based UDP API is similar to the
bind-and-send service we saw earlier, but the main difference is that a
connection is only able to send to the ``remoteAddress`` it was connected to,
and will receive datagrams only from that address.

.. includecode:: code/docs/io/UdpDocTest.java#connected

Consequently the example shown here looks quite similar to the previous one,
the biggest difference is the absence of remote address information in
:meth:`send` and :class:`Received` messages.

.. note::
  
  There is a small performance benefit in using connection based UDP API over
  the connectionless one.  If there is a SecurityManager enabled on the system,
  every connectionless message send has to go through a security check, while
  in the case of connection-based UDP the security check is cached after
  connect, thus writes do not suffer an additional performance penalty.

