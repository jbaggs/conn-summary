..  -*- mode: rst-mode -*-
..

====================================================
conn-summary - Generating network traffic summaries
====================================================

Overview
--------

``conn-summary`` was forked from the Zeek ``trace-summary`` script.
It is meant to work in an environment where Zeek logs are fed into Elasticsearch.
The datastore obviates the need for generating reports at the time of log rotation,
and facilitates generating reports for arbitrary time intervals.
``conn-summary`` is for generating such on-demand reports.

Sample output::

 >== Total === 2020-01-24T00:52:01.708114Z - 2020-01-24T01:51:47.470872Z
   - Connections 43.4k - Payload 398.4m -
     Ports        | Sources                   | Destinations              | Services           | Protocols | States        |
     80     21.7% | 207.240.215.71       3.0% | 239.255.255.253      8.0% | other        51.0% | 17  55.8% | S0      46.2% |
     427    13.0% | 131.243.91.71        2.2% | 131.243.91.255       4.0% | http         21.7% | 6   36.4% | SF      30.1% |
     443     3.8% | 128.3.161.76         1.7% | 131.243.89.138       2.1% | i-echo        7.3% | 1    7.7% | OTH      7.8% |
     138     3.7% | 131.243.90.138       1.6% | 255.255.255.255      1.7% | https         3.8% |           | RSTO     5.8% |
     515     2.4% | 131.243.88.159       1.6% | 128.3.97.204         1.5% | nb-dgm        3.7% |           | SHR      4.4% |
     11001   2.3% | 131.243.88.202       1.4% | 131.243.88.107       1.1% | printer       2.4% |           | REJ      3.0% |
     53      1.9% | 131.243.89.250       1.4% | 117.72.94.10         1.1% | dns           1.9% |           | S1       1.0% |
     161     1.6% | 131.243.89.80        1.3% | 131.243.88.64        1.1% | snmp          1.6% |           | RSTR     0.9% |
     137     1.4% | 131.243.90.52        1.3% | 131.243.88.159       1.1% | nb-ns         1.4% |           | SH       0.3% |
     2222    1.1% | 128.3.161.252        1.2% | 131.243.91.92        1.1% | ntp           1.0% |           | RSTRH    0.2% |


Prerequisites
-------------

* This script requires the Python Elasticsearch Client
  `elasticsearch-py (github) <https://github.com/elastic/elasticsearch-py>`_
  `elasticsearch-py (pypi) <https://pypi.org/project/elasticsearch>`_

* If installed on a system not running Zeek, the `pysubnettree
  <https://github.com/zeek/pysubnettree>`_ package will need to be installed separately.

Installation
------------

* Copy the script into some directory which is in your ``PATH``.
* Edit the ``conn-summary_conf.json`` file to match your environment,
  and place it in ``/usr/local/etc``. If placed elsewhere, you will
  need to edit the line in ``conn-summary`` under ``Path to configuration.``

Configuration 
_____________

Fields in ``conn-summary_conf.json`` that will likely need editing:

* ``"elasticsearch": "host"`` The IP or hostname of your Elasticsearch instance.
* ``"zeek": "index_pattern"`` The pattern to match indices containing Zeek data.
* ``"zeek": "log_stream": "key"`` The key that identifies what Zeek log stream a record is from.
* ``"zeek": "log_stream": "name"`` What the Zeek conn.log  stream is named in Elasticsearch. 
* ``"zeek": "remap"`` Change any fields here that are remapped in your Zeek -> Elasticsearch pipeline.

Options:

* ``"zeekctl": "networks"`` should contain the path to a file designating local networks.
  Eliminating this entry will disable the "Incoming" and "Outgoing" reports.
* ``"zeekctl": "lib_path"`` The path to zeekctl libraries.
  ``conn-summary`` adds this entry to ``PATH`` in order to use the pysubnettree modules
  bundled with zeekctl.  If you have installed pysubnettree manually,
  you can eliminate this entry to prevent ``conn-summary`` from modifying the search path.

Edit ``zeekctl.cfg`` adding ``TraceSummary =`` to disable the summary reports ``zeekctl`` does
with log rotation.

**Note:** ``conn-summary`` expects timestamps in Elasticsearch to be stored in ISO 8601 format,
as it is the only format shared between Zeek and Elasticsearch that does not result in a loss of resolution.

Usage
-----

The general usage is::

   conn-summary [options] | conn-summary [options] < time | interval> |  conn-summary [options] <time> <time>

Run without arguments, ``conn-summary`` will produce a report for the previous 60 minutes to "now".

For periods of 24 hours or less, an  interval in minutes or hours can be provided
by appending "``m``" or "``h``" respectively, to a numeric value.
(e.g. ``conn-summary 120m`` or ``conn-summary 2h`` will produce a report covering the last 2 hours.)

Given a single timestamp, ``conn-summary`` will produce a report from the time provided, to "now".
Provided two timestamps, the report will cover the time range specified.

Timestamps are required to be in ISO 8601 ``%Y-%m-%dT%H:%M:%S.%fZ`` e.g. ``2020-01-23T23:17:05.310878Z`` format.

Options
_______

Run ``conn-summary --help`` for this list of options:

:-b:
    Counts all percentages in bytes rather than number of
    packets/connections.

:-i <duration>:
    Creates totals for each time interval of the given length
    (default is seconds; add "``m``" for minutes and "``h``" for
    hours).

:-n <n>:
    Show top n entries in each break-down. Default is 10.

