# About this repository
Original OFED-1.5 fail to build / install on Ubuntu 13.04 with 2.6.32 kernel.
This repository contains original OFED-1.5 and modifications on it for build /
installation on Ubuntu 13.04 with 2.6.32 kernel.

# Install command
To install, run next commands.
```
$ sudo apt-get install rpm zlib1g-dev libstdc++6-4.7-dev g++ byacc libtool
$ sudo apt-get install bison flex tk-lib tk8.5 tk8.5-dev pciutils
$ sudo cp /bin/sh /bin/sh.bak
$ sudo ln -s /bin/bash /bin/sh
$ sudo ./install.pl -vvv -c ofed-all.conf
```

# Note
This repository does not care MPI. Will not support it.

Not tested from the scratch.

Remember, this is just an **work-around** that worked for our environment. It
may need more effort to success on your system.

# Author
SeongJAe Park(sj38.park@gmail.com)
