Specifications
=============

Version cycle
-------------------

The set of software packages for the solution follows the following rules for naming versions.

The version is divided into 3 digits (A.B.C)
  - A: the 1st digit indicates the major version. Incrementation of this figure implies
     - the addition of major features (with potentially a loss of compatibility with the previous version)
     - adding minor features
     - bug fix
  - B: The 2nd digit indicates a minor version. Incrementing this number indicates
     - adding minor features
     - bug fix
  - C: the 3rd digit indicates a maintenance version. Incrementing this number indicates
     - bug fix

Server tree
-------------------

All files handled by the server are stored in the ``[....]/var/`` directory.

::
  
  scripts/
  serverengine/
  servercontrols/
  serverinterfaces/
  serverrepositories/
  libs/
  testcreatorlib
  testexecutorlib/
  sutadapters/
  var/
    tests/
    testsresult/
    logs/
    tasks/

The tests are stored in the ``[....]/var/tests/`` directory, they are organized by project ID.

Data model
-------------------

A database is used by the server to store:
  - the users of the solution
  - the list of projects
  - test data (project variables)
  - statistics
  - the history of executions

+---------------------------+--------------------------------------------------+
| Tables                    | Description                                      |
+---------------------------+--------------------------------------------------+
| xtc-config                | Server Configuration                             |
+---------------------------+--------------------------------------------------+
| xtc-projects              | List of projects                                 |
+---------------------------+--------------------------------------------------+
| xtc-relations-projects    | Relationship between projects and users          |
+---------------------------+--------------------------------------------------+
| xtc-users                 | List of users                                    |
+---------------------------+--------------------------------------------------+
| xtc-test-environment      | List of variables in JSON format                 |
+---------------------------+--------------------------------------------------+
| xtc-tasks-history         | History of tasks running on the server           |
+---------------------------+--------------------------------------------------+

Passwords management
-------------------

No password (in plain text) is stored in the database. Using a hash is however used.
The hash of the password is stored in the `xtc-users` table.

The algorithm used:

.. code-block::
  
  hash_password = SHA1 ( SALT + SHA1(user_password) )
  

.. image:: /_static/images/server/server_table_pwd.png

File format
-------------------

The tests are in ``XML`` format. There are several test formats:
  - Xml Test Unit
  - Xml Test Suite
  - Xml Test Plan
  - Global Xml Test

**Common XML Structure**

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8" ?>
    <file>
        <properties>
            <descriptions>...</descriptions>
            <inputs-parameters>...</inputs-parameters>
        </properties>
    </file>

**Test Unit Xml**

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8" ?>
    <file>
        <properties>....</properties>
        <testdefinition><![CDATA[pass]]></testdefinition>
        <testdevelopment>1448190694.813723</testdevelopment>
    </file>
    

**Test Suite Xml**

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8" ?>
    <file>
        <properties>...</properties>
        <testdefinition><![CDATA[pass]]></testdefinition>
        <testexecution><![CDATA[pass]]></testexecution>
        <testdevelopment>1448190717.236711</testdevelopment>
    </file>
    

**Test Plan Xml**

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8" ?>
    <file>
        <properties>...</properties>
        <testplan id="0">
            <testfile>
                <id>1</id>
                <color />
                <file>Common:Defaults/testunit.tux</file>
                <enable>2</enable>
                <extension>tux</extension>
                <alias />
                <type>remote</type>
                <parent>0</parent>
                <properties>....</properties>
                <description />
            </testfile>
        </testplan>
        <testdevelopment>1448190725.096519</testdevelopment>
    </file>
    

**Test Global Xml**

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8" ?>
    <file>
        <properties>...</properties>
        <testplan id="0">
            <testfile>
                <id>1</id>
                <color />
                <file>Common:Defaults/testplan.tpx</file>
                <enable>2</enable>
                <extension>tpx</extension>
                <alias />
                <type>remote</type>
                <parent>0</parent>
                <properties>...</properties>
                <description />
            </testfile>
        </testplan>
        <testdevelopment>1448190733.690697</testdevelopment>
    </file>
    

Storage of test results
-------------------------------

The test results are stored on the server in the ``[....]/var/testsresult`` directory.

The results are stored:
  - by the id of the test projects
  - by the date of the day of execution of the test
  - and finally by the date and time of the tests.
 
Organization of the results:

.. code-block:: bash

    Répertoire: <project_id>
        - Répertoire: <yyyy-mm-dd>
            - Répertoire: <yyyy-mm-dd_hh:mm:ss.testid.testname.username>
                - Fichier: TESTPATH 
                - Fichier: test.log
                - Fichier: test.ini
                - Fichier: <testname>_<replayid>.hdr
                - Fichier: <testname>_<replayid>_<result>_<nbcomments>.trv
                - Fichier: <testname>_<replayid>.tbrp
                - Fichier: <testname>_<replayid>.tdsx
                - Fichier: <testname>_<replayid>.trd
                - Fichier: <testname>_<replayid>.trp
                - Fichier: <testname>_<replayid>.trpx
                - Fichier: <testname>_<replayid>.trv
                - Fichier: <testname>_<replayid>.trvx
    

Description of files:

  - ``TESTPATH`` contains the full path for the test result
  - ``test.log`` contains the internal logs of the test, to be used to debug the test framework
  - ``test.ini`` contains test-specific parameters
  - ``<testname>_<replayid>.hdr`` represents the header of the test result
  - ``<testname>_<replayid>_<result>_<nbcomments>.trv`` contains all the events generated during the execution of the tests
  - ``<testname>_<replayid>.tbrp`` contains the basic report in html format
  - ``<testname>_<replayid>.trp`` contains the full report in html
  - ``<testname>_<replayid>.trv`` contains the results report in csv format
  
Control Agents
---------------

The control of the agents since a test is carried out through:
  - the adapters
  - and the server

The communication takes place with the exchange of some specific messages:
  - ``init``: allows to initialize an agent
  - ``notify``: send a message to the agent without waiting for a response
  - ``reset``: allows to reset the agent
  - ``error``: allows the agent to send an error to the adapter
  - ``data``: allows the agent to send data to the adapter

Direction of available communications:
  - Agent -> server -> adapter -> test
  - Test -> adapter -> server -> agent
 
+-----------------------------------+-------------------------------------------------+
|                                   | Agent                                           |
|                                   +-------------------------+-----------------------+
|                                   | Function                | Callback              |
+-----------------------------------+-------------------------+-----------------------+
| Send an error message             | def sendError           |                       |
|                                   | * request               |                       |
|                                   | * data                  |                       |
+-----------------------------------+-------------------------+-----------------------+
| Send a "notify" message           | def sendNotify          |                       |
|                                   | * request               |                       |
|                                   | * data                  |                       |
+-----------------------------------+-------------------------+-----------------------+
| Send a "data" message             | def sendData            |                       |
|                                   | * request               |                       |
|                                   | * data                  |                       |
+-----------------------------------+-------------------------+-----------------------+
| Receiving an "init" message       |                         | def onAgentInit       |
|                                   |                         | * customer            |
|                                   |                         | * tid                 |
|                                   |                         | * request             |
+-----------------------------------+-------------------------+-----------------------+
| Receiving a "reset" message       |                         | def onAgentNotify     |
|                                   |                         | * customer            |
|                                   |                         | * tid                 |
|                                   |                         | * request             |
+-----------------------------------+-------------------------+-----------------------+
| Receiving a "notify" message      |                         | def onAgentReset      |
|                                   |                         | * customer            |
|                                   |                         | * tid                 |
|                                   |                         | * request             |
+-----------------------------------+-------------------------+-----------------------+


+-----------------------------------+------------------------------------------------------------+
|                                   | Adapter                                                    |
|                                   +---------------------------+--------------------------------+
|                                   | Function                  | Callback                       |
+-----------------------------------+---------------------------+--------------------------------+
| Receiving an error message        |                           | def receivedErrorFromAgent     |
|                                   |                           | * data                         |
+-----------------------------------+---------------------------+--------------------------------+
| Receiving a "notify" message      |                           | def receivedNotifyFromAgent    |
|                                   |                           | * data                         |
+-----------------------------------+---------------------------+--------------------------------+
| Receiving a "data" message        |                           | def receivedDataFromAgent      |
|                                   |                           | * data                         |
+-----------------------------------+---------------------------+--------------------------------+
| Send an "init" message            | def initAgent             |                                |
|                                   | * data                    |                                |
+-----------------------------------+---------------------------+--------------------------------+
| Send a "reset" message            | def resetAgent            |                                |
+-----------------------------------+---------------------------+--------------------------------+
| Send a "notify" message           | def sendNotifyToAgent     |                                |
|                                   | * data                    |                                |
+-----------------------------------+---------------------------+--------------------------------+

The server logs
----------------

The server logs are located in the ``[....]/var/logs/`` directory.

+----------------------+--------------------------------------------+
| output.log           | server logs                                |
+----------------------+--------------------------------------------+