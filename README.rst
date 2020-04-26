Micropython MusicPlayerDaemon client module
*******************************************

An MPD (Music Player Daemon) client library written in Micropython.  Forked from [python-musicpd](https://gitlab.com/kaliko/python-musicpd), and should continue to work in mainline Python.

The documentation for this project remains the same as for the original python-musicpd, except for the installation section.  The project must me manually cloned and added to your code manually

----

Installation
=============

Micropython 1.9+
----------------
git clone git+https://github.com/jockm/upython-musicpd

Manually add to project

Python 3.4+
-----------
**Latest development version** using pip + git:

.. code:: bash

    pip install git+https://github.com/jockm/upython-musicpd


Library overview
----------------
Here is a snippet allowing to list the last modified artists in the media library:

.. code:: python3

        #!/usr/bin/env python3
        # coding: utf-8
        import musicpd

        def main():
            cli = musicpd.MPDClient()
            cli.connect()
            # Gets files count in the library
            nb_files = int(cli.stats()['songs'])
            # Gets the last 100 files modified
            files = cli.search('file', '', 'sort', 'Last-Modified', 'window', (nb_files-100,))
            # Print last modified artists in media library
            print(' / '.join({f.get('albumartist') for f in files}))
            cli.disconnect()

        # Script starts here
        if __name__ == '__main__':
            main()


Build documentation
--------------------

.. code:: bash

    # Get the source
    git clone https://gitlab.com/kaliko/python-musicpd.git && cd python-musicpd
    # Installs sphinx if needed
    python3 -m venv venv && . ./venv/bin/activate
    pip install sphinx
    # And build
    python3 setup.py build_sphinx
    # Or call sphinx
    sphinx-build -d ./doc/build/doctrees doc/source -b html ./doc/build/html

Using the client library
=========================

The client library can be used as follows:

.. code-block:: python

    client = musicpd.MPDClient()       # create client object
    client.connect()                   # use MPD_HOST/MPD_PORT if set else
                                       #   test ${XDG_RUNTIME_DIR}/mpd/socket for existence
                                       #   fallback to localhost:6600
                                       # connect support host/port argument as well
    print(client.mpd_version)          # print the mpd protocol version
    print(client.cmd('one', 2))        # print result of the command "cmd one 2"
    client.disconnect()                # disconnect from the server

For a list of supported commands, their arguments (as MPD currently understands
them), and the functions used to parse their responses see commands

See the `MPD protocol documentation`_ for more details.

Command lists are also supported using `command_list_ok_begin()` and
`command_list_end()` :

.. code-block:: python

    client.command_list_ok_begin()       # start a command list
    client.update()                      # insert the update command into the list
    client.status()                      # insert the status command into the list
    results = client.command_list_end()  # results will be a list with the results

Provide a 2-tuple as argument for command supporting ranges (cf. `MPD protocol documentation`_ for more details).
Possible ranges are: "START:END", "START:" and ":" :

.. code-block:: python

    # An intelligent clear
    # clears played track in the queue, currentsong included
    pos = client.currentsong().get('pos', 0)
    # the 2-tuple range object accepts str, no need to convert to int
    client.delete((0, pos))
    # missing end interpreted as highest value possible, pay attention still need a tuple.
    client.delete((pos,))  # purge queue from current to the end

A notable case is the `rangeid` command allowing an empty range specified
as a single colon as argument (i.e. sending just ":"):

.. code-block:: python

    # sending "rangeid :" to clear the range, play everything
    client.rangeid(())  # send an empty tuple

Empty start in range (i.e. ":END") are not possible and will raise a CommandError.


Commands may also return iterators instead of lists if `iterate` is set to
`True`:

.. code-block:: python

    client.iterate = True
    for song in client.playlistinfo():
        print song['file']

Each command have a *send\_<CMD>* and a *fetch\_<CMD>* variant, which allows to
send a MPD command and then fetch the result later.
This is useful for the idle command:

.. code-block:: python

    >>> client.send_idle()
    # do something else or use function like select()
    # http://docs.python.org/howto/sockets.html#non-blocking-sockets
    # ex. select([client], [], [])
    >>> events = client.fetch_idle()

    # more complex use for example, with glib/gobject:
    >>> def callback(source, condition):
    >>>    changes = client.fetch_idle()
    >>>    print changes
    >>>    return False  # removes the IO watcher

    >>> client.send_idle()
    >>> gobject.io_add_watch(client, gobject.IO_IN, callback)
    >>> gobject.MainLoop().run()


.. _MPD protocol documentation: http://www.musicpd.org/doc/protocol/

.. _commands:

Available commands
==================

Get current available commands:

.. code-block:: python

   import musicpd
   print(' '.join([cmd for cmd in musicpd.MPDClient()._commands.keys()]))


List, last updated for v0.4.4:

Status Commands
------------------
+-----------------+-----------------------+------------------+
| Command         | Arguments             | Returns          |
+=================+=======================+==================+
|clearerror       | (none)                |  (nothing)       |
+-----------------+-----------------------+------------------+
|currentsong      | (none)                |  fetch_object    |
+-----------------+-----------------------+------------------+
|idle             | str                   |  fetch_list      |
+-----------------+-----------------------+------------------+
|noidle           | (none)                |  None            |
+-----------------+-----------------------+------------------+
|status           | (none)                |  fetch_object    |
+-----------------+-----------------------+------------------+
|stats            | (none)                |  fetch_object    |
+-----------------+-----------------------+------------------+

Playback Option Commands
------------------------
+------------------+-----------------------+------------------+
| Command          | Arguments             | Returns          |
+==================+=======================+==================+
|consume           | bool                  |  (nothing)       |
+------------------+-----------------------+------------------+
|crossfade         | int                   |  (nothing)       |
+------------------+-----------------------+------------------+
|mixrampdb         | str                   |  (nothing)       |
+------------------+-----------------------+------------------+
|mixrampdelay      | int                   |  (nothing)       |
+------------------+-----------------------+------------------+
|random            | bool                  |  (nothing)       |
+------------------+-----------------------+------------------+
|repeat            | bool                  |  (nothing)       |
+------------------+-----------------------+------------------+
|setvol            | int                   |  (nothing)       |
+------------------+-----------------------+------------------+
|single            | bool                  |  (nothing)       |
+------------------+-----------------------+------------------+
|replay_gain_mode  | str                   |  (nothing)       |
+------------------+-----------------------+------------------+
|replay_gain_status| (none)                |  fetch_item      |
+------------------+-----------------------+------------------+
|volume            |int                    |  (nothing)       |
+------------------+-----------------------+------------------+


Playback Control Commands
-------------------------
+-----------------+--------------------+------------------+
| Command         | Arguments          | Returns          |
+=================+====================+==================+
|next             | (none)             | (nothing)        |
+-----------------+--------------------+------------------+
|pause            | bool               | (nothing)        |
+-----------------+--------------------+------------------+
|play             | int                | (nothing)        |
+-----------------+--------------------+------------------+
|playid           | int                | (nothing)        |
+-----------------+--------------------+------------------+
|previous         | (none)             | (nothing)        |
+-----------------+--------------------+------------------+
|seek             | int int            | (nothing)        |
+-----------------+--------------------+------------------+
|seekid           | int int            | (nothing)        |
+-----------------+--------------------+------------------+
|seekcur          | int                | (nothing)        |
+-----------------+--------------------+------------------+
|stop             | (none)             | (nothing)        |
+-----------------+--------------------+------------------+

Queue Commands
--------------
+-----------------+--------------------+------------------+
| Command         | Arguments          | Returns          |
+=================+====================+==================+
|add              | str                | (nothing)        |
+-----------------+--------------------+------------------+
|addid            | str int (Opt)      | fetch_item       |
+-----------------+--------------------+------------------+
|clear            | (none)             | (nothing)        |
+-----------------+--------------------+------------------+
|delete           | int|range          | (nothing)        |
+-----------------+--------------------+------------------+
|deleteid         | int                | (nothing)        |
+-----------------+--------------------+------------------+
|move             | int|range int      | (nothing)        |
+-----------------+--------------------+------------------+
|moveid           | int int            | (nothing)        |
+-----------------+--------------------+------------------+
|playlist         | (none)             | fetch_playlist   |
+-----------------+--------------------+------------------+
|playlistfind     | locate             | fetch_songs      |
+-----------------+--------------------+------------------+
|playlistid       | int (Opt)          | fetch_songs      |
+-----------------+--------------------+------------------+
|playlistinfo     | int|range (Opt)    | fetch_songs      |
+-----------------+--------------------+------------------+
|playlistsearch   | locate             | fetch_songs      |
+-----------------+--------------------+------------------+
|plchanges        | int                | fetch_songs      |
+-----------------+--------------------+------------------+
|plchangesposid   | int                | fetch_changes    |
+-----------------+--------------------+------------------+
|prio             | int int|range      | (nothing)        |
+-----------------+--------------------+------------------+
|prioid           | int int            | (nothing)        |
+-----------------+--------------------+------------------+
|rangeid          | int range          | (nothing)        |
+-----------------+--------------------+------------------+
|shuffle          | range (Opt)        | (nothing)        |
+-----------------+--------------------+------------------+
|swap             | int int            | (nothing)        |
+-----------------+--------------------+------------------+
|swapid           | int int            | (nothing)        |
+-----------------+--------------------+------------------+
|addtagid         | int str str        | (nothing)        |
+-----------------+--------------------+------------------+
|cleartagid       | int str (Opt)      | (nothing)        |
+-----------------+--------------------+------------------+

Stored Playlist Commands
------------------------
+-----------------+--------------------+------------------+
| Command         | Arguments          | Returns          |
+=================+====================+==================+
|listplaylist     | str                | fetch_list       |
+-----------------+--------------------+------------------+
|listplaylistinfo | str                | fetch_songs      |
+-----------------+--------------------+------------------+
|listplaylists    | (none)             | fetch_playlists  |
+-----------------+--------------------+------------------+
|load             | str range (Opt)    | (nothing)        |
+-----------------+--------------------+------------------+
|playlistadd      | str str            | (nothing)        |
+-----------------+--------------------+------------------+
|playlistclear    | str                | (nothing)        |
+-----------------+--------------------+------------------+
|playlistdelete   | str int            | (nothing)        |
+-----------------+--------------------+------------------+
|playlistmove     | str int int        | (nothing)        |
+-----------------+--------------------+------------------+
|rename           | str str            | (nothing)        |
+-----------------+--------------------+------------------+
|rm               | str                | (nothing)        |
+-----------------+--------------------+------------------+
|save             | str                | (nothing)        |
+-----------------+--------------------+------------------+

Database Commands
-----------------
+-----------------+----------------------------------------+----------------+
| Command         | Arguments                              | Returns        |
+=================+========================================+================+
|albumart         | str int                                | fetch_object   |
+-----------------+----------------------------------------+----------------+
|count            | str str                                | fetch_object   |
+-----------------+----------------------------------------+----------------+
|count            | group str                              | fetch_object   |
+-----------------+----------------------------------------+----------------+
|find             | str str str str (Opt)...               | fetch_songs    |
+-----------------+----------------------------------------+----------------+
|findadd          | str str str str (Opt)                  | (nothing)      |
+-----------------+----------------------------------------+----------------+
|list             | str str str (Opt)...group str (Opt)... | fetch_list     |
+-----------------+----------------------------------------+----------------+
|listall          | str (Opt)                              | fetch_database |
+-----------------+----------------------------------------+----------------+
|listallinfo      | str (Opt)                              | fetch_database |
+-----------------+----------------------------------------+----------------+
|listfiles        | str                                    | fetch_database |
+-----------------+----------------------------------------+----------------+
|lsinfo           | str (Opt)                              | fetch_database |
+-----------------+----------------------------------------+----------------+
|readcomments     | str (Opt)                              | fetch_object   |
+-----------------+----------------------------------------+----------------+
|search           | str str str str (Opt)...               | fetch_song     |
+-----------------+----------------------------------------+----------------+
|searchadd        | str str str str (Opt)...               | (nothing)      |
+-----------------+----------------------------------------+----------------+
|searchaddpl      | str str str str str (Opt)...           | (nothing)      |
+-----------------+----------------------------------------+----------------+
|update           | str (Opt)                              | fetch_item     |
+-----------------+----------------------------------------+----------------+
|rescan           | str (Opt)                              | fetch_item     |
+-----------------+----------------------------------------+----------------+

Mounts and neighbors
---------------------
+-----------------+--------------------+------------------+
| Command         | Arguments          | Returns          |
+=================+====================+==================+
|mount            | str str            | (nothing)        |
+-----------------+--------------------+------------------+
|unmount          | str                | (nothing)        |
+-----------------+--------------------+------------------+
|listmounts       | (none)             | fetch_mounts     |
+-----------------+--------------------+------------------+
|listneighbors    | (none)             | fetch_neighbors  |
+-----------------+--------------------+------------------+

Sticker Commands
----------------
+-----------------+--------------------+------------------+
| Command         | Arguments          | Returns          |
+=================+====================+==================+
|sticker   get    | str str str        | fetch_item       |
+-----------------+--------------------+------------------+
|sticker   set    | str str str str    | (nothing)        |
+-----------------+--------------------+------------------+
|sticker   delete | str str str (Opt)  | (nothing)        |
+-----------------+--------------------+------------------+
|sticker   list   | str str            | fetch_list       |
+-----------------+--------------------+------------------+
|sticker   find   | str str str        | fetch_songs      |
+-----------------+--------------------+------------------+

Connection Commands
-------------------                                       
+------------------+--------------------+------------------+
| Command          | Arguments          | Returns          |
+==================+====================+==================+
|close             | (none)             | None             |
+------------------+--------------------+------------------+
|kill              | (none)             | None             |
+------------------+--------------------+------------------+
|password          |  str               | (nothing)        |
+------------------+--------------------+------------------+
|ping              | (none)             | (nothing)        |
+------------------+--------------------+------------------+
|tagtypes          | (none)             | fetch_list       |
+------------------+--------------------+------------------+
|tagtypes disable: | str str (Opt)...   | (nothing)        |
+------------------+--------------------+------------------+
|tagtypes enable:  | str str (Opt)...   | (nothing)        |
+------------------+--------------------+------------------+
|tagtypes clear:   | (none)             | (nothing)        |
+------------------+--------------------+------------------+
|tagtypes all:     | (none)             | (nothing)        |
+------------------+--------------------+------------------+

Partition Commands
------------------
+------------------+--------------------+------------------+
| Command          | Arguments          | Returns          |
+==================+====================+==================+
|partition         | str                | (nothing)        |
+------------------+--------------------+------------------+
|listpartitions    | (none)             | fetch_list       |
+------------------+--------------------+------------------+
|newpartition      | str                | (nothing)        |
+------------------+--------------------+------------------+
                                                           
Audio Output Commands
---------------------
+------------------+--------------------+------------------+
| Command          | Arguments          | Returns          |
+==================+====================+==================+
|disableoutput     | int                | (nothing)        |
+------------------+--------------------+------------------+
|enableoutput      | int                | (nothing)        |
+------------------+--------------------+------------------+
|toggleoutput      | int                | (nothing)        |
+------------------+--------------------+------------------+
|outputs           | (none)             | fetch_outputs    |
+------------------+--------------------+------------------+
|outputset         | str str str        | (nothing)        |
+------------------+--------------------+------------------+

Reflection Commands
-------------------
+------------------+--------------------+------------------+
| Command          | Arguments          | Returns          |
+==================+====================+==================+
| config           | (none)             | fetch_object     |
+------------------+--------------------+------------------+
| commands         | (none)             | fetch_list       |
+------------------+--------------------+------------------+
| notcommands      | (none)             | fetch_list       |
+------------------+--------------------+------------------+
| urlhandlers      | (none)             | fetch_list       |
+------------------+--------------------+------------------+
| decoders         | (none)             | fetch_plugins    |
+------------------+--------------------+------------------+

----

:Documentation: https://kaliko.gitlab.io/python-musicpd
:MPD Protocol:  https://www.musicpd.org/doc/html/protocol.html
:Code:          https://github.com/jockm/upython-musicpd
:Dependencies:  None
:Compatibility: Micropython 1.9+ / Python 3.4+
:Licence:       GNU LGPLv3
