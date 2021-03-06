.. _learn-features:

Learn Device Features
=====================
This topic describes how to use the ``learn`` function of the |librarybold| ``Ops`` module for stateful network validation of device features, such as protocols, interfaces, line cards, and other hardware.

.. include:: ../definitions/def_feature.rst 
   :start-line: 3

.. _cli-learn:

"Learning" Mechanism Explained
------------------------------
.. include:: ../definitions/def_ops.rst
   :start-line: 3

The output is stored with the same :term:`key-value pair` structure across devices. The stored output makes it possible for you to take a snapshot of the network state at different points in time, and then to :ref:`compare network states <compare-network-states>`.

.. tip:: Why use ``learn`` instead of a :term:`parser`? The parsed output for different devices can result in different data structures. The ``learn`` function, by contrast, results in a *consistent* set of keys, which means that you can write *one* script that works on different devices.

.. _supported-features:

Supported Features
------------------

To see a complete list of the features that the |library| can learn, and to see the resulting data structure for each feature, visit the `Models <https://pubhub.devnetcloud.com/media/genie-feature-browser/docs/#/models>`_ page.

If you try to "learn" a feature that we don't yet support, the system returns the following exception:

 .. code-block:: text

    |---------------------------------------------------------------------------------|
    |  Could not learn feature 'acl'                                                  |
    |  -  Exception:      genie_learn_all/acl_nxos_nx-osv-1_exception.txt             |
    |  -  Feature not yet developed for this os                                       |
    |---------------------------------------------------------------------------------|

If you want to request support for a new feature, please contact us at pyats-support-ext@cisco.com

"Learn" Examples
----------------
This topic describes how you can tell the system to learn one or more features.

.. attention:: Before you try these examples, make sure that you :download:`download and extract the zip file <mock.zip>` that contains the mock data and Python script.

Learn a Single Feature
^^^^^^^^^^^^^^^^^^^^^^
To learn one feature on a single device, you can use the device hostname or the device alias (defined in the testbed YAML file). In the following example, ``uut`` is the alias "unit under test" for the host ``nx-osv-1``.

#. In your virtual environment, change to the directory that contains the mock YAML file::

    (pyats) $ cd mock

#. You can use a Python interpreter or the :term:`library command line`.

    * If you want to use Python, you can use ``pyats shell`` to load the ``testbed`` API and create your testbed and device objects. Then, connect to the device and tell the system to learn the feature. In this example, the system stores the output as a Python dictionary in the variable ``output``:

       .. code-block:: 

          (pyats) $ pyats shell --testbed-file mock.yaml
            >>> dev = testbed.devices['uut']
            >>> dev.connect()
            >>> output = dev.learn('ospf')

      *Result*: The system displays a summary of the parsed ``show`` commands that ran. |br| |br| 

    * If you want to use the CLI::

      (pyats) $ pyats learn ospf --testbed-file mock.yaml --devices uut --output output_folder

      *Result*: The system connects to the device, runs the show commands, stores the output in JSON format in the specified directory, and displays a "Learn Summary" that shows the names of the output files. These include:
        
          * Connection log
          * Structured JSON output
          * Device console output |br| |br| 

          .. code-block:: text

                +==============================================================================+
                | Genie Learn Summary for device nx-osv-1                                      |
                +==============================================================================+
                |  Connected to nx-osv-1                                                       |
                |  -   Log: output_folder/connection_uut.txt                                   |
                |------------------------------------------------------------------------------|
                |  Learnt feature 'ospf'                                                       |
                |  -  Ops structure:  output_folder/ospf_nxos_nx-osv-1_ops.txt                 |
                |  -  Device Console: output_folder/ospf_nxos_nx-osv-1_console.txt             |
                |==============================================================================|    

      To see the structured data, use a text editor to open the file :monospace:`output_folder/ospf_nxos_nx-osv-1_ops.txt`.



Learn Multiple Features
^^^^^^^^^^^^^^^^^^^^^^^
You can use the ``learn`` function to get the operational states of multiple or all features, as shown in the following examples.

Across Multiple Devices
"""""""""""""""""""""""
This example shows you how to learn the ``bgp`` and ``ospf`` features on all of the devices in your testbed.

      .. note:: The mock data only contains one device, so you will only see the results for that device.

#. In your virtual environment, change to the directory that contains the mock YAML file::

    (pyats) $ cd mock
    
#. You can use a Python interpreter or the :term:`library command line`.

    * If you want to use Python, you can use ``pyats shell`` to load the ``testbed`` API and create your testbed and device objects. Then, tell the system to connect to each device and to learn the specified features. In this example, the system stores the output as a Python dictionary in the variable ``learnt`` and displays the output::

       (pyats) $ pyats shell --testbed-file mock.yaml
          >>> learnt = {}
          >>> for name, dev in testbed.devices.items():
          ...     dev.connect()
          ...     learnt[name] = {}
          ...     learnt[name]['bgp'] = dev.learn('bgp')
          ...     learnt[name]['ospf'] = dev.learn('ospf')
          ...

      This example uses a Python ``for`` loop to execute each statement on all devices in the testbed. The system stores the feature information in Python dictionaries, each identified by the device name. |br| |br| 

    * If you want to use the CLI::

       (pyats) $ pyats learn bgp ospf --testbed-file mock.yaml --output output_folder

      *Result*: Within the output directory, the system creates the output files and displays a summary for each device.

      The following example shows what you would see.

      .. code-block:: text

        +==============================================================================+
        | Genie Learn Summary for device nx-osv-1                                      |
        +==============================================================================+
        |  Connected to nx-osv-1                                                       |
        |  -   Log: output_folder/connection_nx-osv-1.txt                              |
        |------------------------------------------------------------------------------|
        |  Learnt feature 'bgp'                                                        |
        |  -  Ops structure:  output_folder/bgp_nxos_nx-osv-1_ops.txt                  |
        |  -  Device Console: output_folder/bgp_nxos_nx-osv-1_console.txt              |
        |------------------------------------------------------------------------------|
        |  Learnt feature 'ospf'                                                       |
        |  -  Ops structure:  output_folder/ospf_nxos_nx-osv-1_ops.txt                 |
        |  -  Device Console: output_folder/ospf_nxos_nx-osv-1_console.txt             |
        |==============================================================================|


        +==============================================================================+
        | Genie Learn Summary for device csr1000v-1                                    |
        +==============================================================================+
        |  Connected to csr1000v-1                                                     |
        |  -   Log: output_folder/connection_csr1000v-1.txt                            |
        |------------------------------------------------------------------------------|
        |  Learnt feature 'bgp'                                                        |
        |  -  Ops structure:  output_folder/bgp_iosxe_csr1000v-1_ops.txt               |
        |  -  Device Console: output_folder/bgp_iosxe_csr1000v-1_console.txt           |
        |------------------------------------------------------------------------------|
        |  Learnt feature 'ospf'                                                       |
        |  -  Ops structure:  output_folder/ospf_iosxe_csr1000v-1_ops.txt              |
        |  -  Device Console: output_folder/ospf_iosxe_csr1000v-1_console.txt          |
        |==============================================================================|


      To see the structured data, use a text editor to open any of the ``ops.txt`` files.

On a Single Device
""""""""""""""""""

.. tip:: Use the ``learn all`` functionality to learn all of the supported features on a device. The system returns the results in a format with key-value pairs, and notifies you of any exceptions for features it did not learn.

#. In your virtual environment, change to the directory that contains the mock YAML file::

    (pyats) $ cd mock
    
#. You can use a Python interpreter or the :term:`library command line`.

    * If you want to use Python, you can use ``pyats shell`` to load the ``testbed`` API and create your testbed and device objects. Then, connect to the device and tell the system to learn all of the features. In this example, the system stores the output as a Python dictionary in the variable ``output``::

       (pyats) $ pyats shell --testbed-file mock.yaml
          >>> dev = testbed.devices['uut']
          >>> dev.connect()
          >>> output = dev.learn('all')

      .. the dev.learn('all') didn't work for me

    * If you want to use the CLI::

      (pyats) $ pyats learn all --testbed-file mock.yaml --devices uut --output output_folder

      *Result*: The system saves all of the console and structured output files in JSON format to the specified directory and displays a summary of the results, as shown in the following example. 

      .. code-block:: text

        +=================================================================================+
        | Genie Learn Summary for device nx-osv-1                                         |
        +=================================================================================+
        |  Connected to nx-osv-1                                                          |
        |  -   Log: genie_learn_all/connection_uut.txt                                    |
        |---------------------------------------------------------------------------------|
        |  Could not learn feature 'acl'                                                  |
        |  -  Exception:      genie_learn_all/acl_nxos_nx-osv-1_exception.txt             |
        |  -  Feature not yet developed for this os                                      |
        |---------------------------------------------------------------------------------|
        |  Learnt feature 'bgp'                                                           |
        |  -  Ops structure:  genie_learn_all/bgp_nxos_nx-osv-1_ops.txt                   |
        |  -  Device Console: genie_learn_all/bgp_nxos_nx-osv-1_console.txt               |
        |---------------------------------------------------------------------------------|
        |  Could not learn feature 'dot1x'                                                |
        |  -  Exception:      genie_learn_all/dot1x_nxos_nx-osv-1_exception.txt           |
        |  -  Feature not yet developed for this os                                       |
        |---------------------------------------------------------------------------------|

      To see the structured data, use a text editor to open any of the ``ops.txt`` files.

.. note:: If we don't support a feature on a device, the system returns an exception, as shown in this example. For more information, see the section :ref:`supported-features`.

See also...

* `Description of the Ops module <https://pubhub.devnetcloud.com/media/genie-docs/docs/userguide/Ops/index.html#ops-guide>`_








