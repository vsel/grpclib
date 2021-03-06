**WARNING**: this library is a prototype, under active development.

This project is a pure-Python `gRPC`_ implementation, based on `hyper-h2`_
project, **requires Python >= 3.5**.

Motivation: ``grpclib`` is intended to implement gRPC protocol in Python once
and natively support all concurrency models. However, currently ``grpclib``
supports only ``asyncio`` library and only with ``async/await`` syntax, because
this is the most modern for Python stack, which was not supported yet.

Note: Python 2.7 support is not planned, but you can use official `grpcio`_
library for projects with such requirements.

Installation
~~~~~~~~~~~~

Currently you can install ``grpclib`` only from the GitHub:

.. code-block:: shell

    $ pip3 install git+https://github.com/vmagamedov/grpclib.git

For the code generation you will also need a ``protoc`` compiler, which can be
installed with ``protobuf`` package:

.. code-block:: shell

    $ brew install protobuf  # example for macOS users

Or you can use ``protoc`` compiler from the ``grpcio-tools`` Python package:

.. code-block:: shell

    $ pip3 install grpcio-tools

**Note**: ``grpcio`` and ``grpcio-tools`` packages are **not required in
runtime**, ``grpcio-tools`` package will be used only during code generation.

Protoc plugin
~~~~~~~~~~~~~

In order to use this library you will have to generate special stub files using
plugin provided, which can be used like this:

.. code-block:: shell

    $ python3 -m grpc_tools.protoc -I. --python_out=. --python_grpc_out=. helloworld.proto

This command will generate ``helloworld_pb2.py`` and ``helloworld_grpc.py``
files.

Plugin, which implements ``--python_grpc_out`` option is available for
``protoc`` compiler as ``protoc-gen-python_grpc`` executable, which will be
installed by ``setuptools`` into your ``PATH`` during installation of the
``grpclib`` library.

Example
~~~~~~~

See ``example`` directory for a full example of the ``helloworld`` service.
``example/README.rst`` contains instructions about how to generate
``helloworld_pb2.py`` and ``helloworld_grpc.py`` files and how to run example.

Example basically looks like this:

.. code-block:: python

    import asyncio

    from grpclib.server import Server
    from grpclib.client import Channel

    from .helloworld_pb2 import HelloRequest, HelloReply
    from .helloworld_grpc import GreeterBase, GreeterStub

    loop = asyncio.get_event_loop()

    # Server

    class Greeter(GreeterBase):

        async def SayHello(self, stream):
            request = await stream.recv_message()
            message = 'Hello, {}!'.format(request.name)
            await stream.send_message(HelloReply(message=message))

    server = Server([Greeter()], loop=loop)
    loop.run_until_complete(server.start('127.0.0.1', 50051))

    # Client

    channel = Channel(loop=loop)
    stub = GreeterStub(channel)

    async def make_request():
        response = await stub.SayHello(HelloRequest(name='World'))
        assert response.message == 'Hello, World!'

    # Test request

    loop.run_until_complete(make_request())

    # Shutdown

    server.close()
    loop.run_until_complete(server.wait_closed())
    loop.close()

Where ``helloworld.proto`` contains:

.. code-block:: protobuf

    syntax = "proto3";

    package helloworld;

    service Greeter {
      rpc SayHello (HelloRequest) returns (HelloReply) {}
    }

    message HelloRequest {
      string name = 1;
    }

    message HelloReply {
      string message = 1;
    }

Changelog
~~~~~~~~~

* ``0.2.0`` - complete rewrite, pure-Python, based on `hyper-h2`_
* ``0.1.0`` – workaround implemented, only server implementation available and
  only for unary calls

.. _gRPC: http://www.grpc.io
.. _hyper-h2: https://github.com/python-hyper/hyper-h2
.. _grpcio: https://pypi.org/project/grpcio/
