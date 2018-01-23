===================
Vagga in VirtualBox
===================

:Status: PoC

This is a prototype which brings vagga as the first-class tool to OS X and
(possibly) windows using virtualbox


Installation
============

First `download and install virtualbox`__. The project is tested on
VirtualBox 5.1 but may work on earlier versions too.

Then run the following commands (assuming you have brew_ installed)::

    $ brew install python3 unison wget
    $ pip3 install git+http://github.com/tailhook/vagga-box
    [ .. snip .. ]
    $ vagga
    Available commands:
        run

Effectively it requires python >= 3.5 and unison 2.48.4 (unison is very picky
on version numbers)

__ https://www.virtualbox.org/wiki/Downloads
.. _brew: http://brew.sh

IDE support is enabled by the following command (and requires sudo access)::

    $ vagga _box mount
    Running sudo mount -t nfs -o vers=4,resvport,port=7049 127.0.0.1:/vagga /Users/myuser/.vagga/remote
    Password:
    Now you can add ~/.vagga/remote/<project-name>/.vagga/<container-name>/dir
    to the search paths of your IDE

You need to run it each time your machine is rebooted, or if you restarted your
virtualbox manually.


Upgrading
=========

Once you have installed vagga-box you can upgrade vagga inside the container
using the following command-line::

    vagga _box upgrade_vagga

Changing the disk size
======================

By default, the disk size in Virtualbox is set to 20 GB. If necessary, it can be increased by the following steps:

1. Change VM's image size::

        $ vagga _box down
        $ VBoxManage modifyhd ~/.vagga/vm/storage.vdi --resize 40860
        $ vagga _box up

2. Change partition size inside VM::

        $ vagga _box ssh
        $ sudo apk add cfdisk e2fsprogs-extra
        $ sudo cfdisk /dev/sdb
            [ .. Delete /dev/sdb1 ..]
            [ .. New partition / default size / Primary .. ]
            [ .. Write changes / Quit .. ]
        $ sudo reboot

3. Final steps::

        $ vagga _box ssh
        $ sudo resize2fs /dev/sdb1
        $ sudo df -h  # Check size

Short FAQ
=========

**Why is it in python?** For a quick prototype. It will be integrated into
vagga as soon as is proven to be useful. Or may be we leave it in python if
it would be easier to install.

**So should I try this version or wait it integrated in vagga?** Definitely you
should try. The integrated version will work the same.

**Is there any difference between this and vagga on linux?** There are two key
differences:

* you need to export ports that you want to be accessible from the
  host system
* we keep all the container files (and a copy of the project) in the virtualbox
* to view it from the host system mount nfs volume (``vagga _box mount``)
* to make filesync fast you can add some dirs to the ignore list
  (``_ignore-dirs`` setting)

.. code-block:: yaml

    _ignore-dirs:
    - .git
    - tmp
    - data

    containers:
      django:
        setup:
        - !Alpine v3.3
        - !Py3Install ['Django >=1.9,<1.10']

    commands:
      run: !Command
        description: Start the django development server
        container: django
        _expose-ports: [8080]
        run: python3 manage.py runserver

**Please report if you find any other differences using the tool**. Ah, but
exact text of some error messages may differ, don't be too picky :)

**Why `_expose-ports` are underscored?** This is a standard
way to add extension metadata or user-defined things in vagga.yaml. We will
remove the underscore as soon as integrate it into main code. Fixing
underscores isn't going to be a big deal.

**Will linux users add `_expose-ports` for me?** Frankly,
currently probably now. But it's small change that probably no one will need
to delete. In the future we want to apply ``seccomp`` filters to allow to bind
only exposed ports on linux too.

**What will be changed when we integrate this into vagga?** We will move more
operations from virtualbox into host system. For example list of commands will
be executed by mac os. Also ``vagga _list``, some parts of ``vagga _clean`` and
so on. But we will do our best to keep semantics exactly the same.


LICENSE
=======

This project has been placed into the public domain.
