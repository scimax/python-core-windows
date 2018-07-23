# python-core-windows
Windows Server Core container running python 3.6.0 and pip 9.0.1. The docker image is based on the [Windows Nano Server for python](https://github.com/LBates2000/python-windows) by lbates2000. The nano server was replaced with the server core container as the [ctypes](https://docs.python.org/3/library/ctypes.html) library for foreign functions does not work properly.

# Issue with the nano server
The following snippet is a minimum code required to reproduce the problem based on `__init__.py` in the [comtypes](https://github.com/enthought/comtypes/blob/36e997b11cbfd9fe755b0dc58723eb915edcc393/comtypes/__init__.py#L156) module:
```py
from ctypes import *
import sys
COINIT_APARTMENTTHREADED = 0x2
_ole32 = oledll.ole32
flags = getattr(sys, "coinit_flags", COINIT_APARTMENTTHREADED)
_ole32.CoInitializeEx(None, flags)
```
The traceback reports
```py
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "_ctypes/callproc.c", line 918, in GetResult
OSError: [WinError -2147467263] Not implemented
```

Unfortunately, using the windows core server instead of the nano server blows up the size of the container by *10GB*. But ctypes works with this container base.

# Build
After cloning the repository from [github](https://github.com/scimax/python-core-windows) run:
```
docker build -t python-core-windows .
```


# Usage
This was built by installing Python on a Windows 10 x64 machine and then copying the installation to the container and setting the python path. 

You can run it interactively
```
docker run -it python-windows
```
or you can use it as a base for another container, as well as upgrade PIP and install additional Python modules, by adding the appropriate commands to the new container's Dockerfile.

For example:
```
FROM python-windows:latest

ENV APP_DIR C:/apps
ADD requirements.txt $APP_DIR/requirements.txt
WORKDIR $APP_DIR

RUN \
	python -m pip install --upgrade pip && \
	python -m pip install -r requirements.txt

ENTRYPOINT ["C:\\Python\\python.exe"]
```

Container is hosted on Docker Hub:

https://hub.docker.com/r/scimax/python-core-windows/

```
docker pull scimax/python-core-windows
```

