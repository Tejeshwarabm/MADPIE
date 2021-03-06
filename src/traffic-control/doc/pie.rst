.. include:: replace.txt
.. highlight:: cpp

PIE queue disc
----------------

This chapter describes the PIE ([Pan13]_, [Pan16]_) queue disc implementation 
in |ns3|. 

Proportional Integral controller Enhanced (PIE) is a queuing discipline that aims to
solve the bufferbloat [Buf14]_ problem. The model in ns-3 is a port of Preethi
Natarajan's ns-2 PIE model.

MADPIE extension of PIE
-----------------------

Maximum and Average queuing Delay with Proportional Integral controller Enhanced (MADPIE)
[Kuhn16]_ is an extension of PIE, that adds deterministic packet drops at controlled
intervals.

Model Description
*****************

The source code for the PIE model is located in the directory ``src/traffic-control/model``
and consists of 2 files `pie-queue-disc.h` and `pie-queue-disc.cc` defining a PieQueueDisc
class. The code was ported to |ns3| by Mohit P. Tahiliani, Shravya K. S. and Smriti Murali
based on ns-2 code implemented by Preethi Natarajan, Rong Pan, Chiara Piglione, Greg White
and Takashi Hayakawa.  

* class :cpp:class:`PieQueueDisc`: This class implements the main PIE algorithm:

  * ``PieQueueDisc::DoEnqueue ()``: This routine checks whether the queue is full, and if so, drops the packets and records the number of drops due to queue overflow. If queue is not full, this routine calls ``PieQueueDisc::DropEarly()``, and depending on the value returned, the incoming packet is either enqueued or dropped.

  * ``PieQueueDisc::DropEarly ()``: The decision to enqueue or drop the packet is taken by invoking this routine, which returns a boolean value; false indicates enqueue and true indicates drop.

  * ``PieQueueDisc::CalculateP ()``: This routine is called at a regular interval of `m_tUpdate` and updates the drop probability, which is required by ``PieQueueDisc::DropEarly()``

  * ``PieQueueDisc::DoDequeue ()``: This routine calculates the average departure rate which is required for updating the drop probability in ``PieQueueDisc::CalculateP ()``  

References
==========

.. [Pan13] Pan, R., Natarajan, P., Piglione, C., Prabhu, M. S., Subramanian, V., Baker, F., & VerSteeg, B. (2013, July). PIE: A lightweight control scheme to address the bufferbloat problem. In High Performance Switching and Routing (HPSR), 2013 IEEE 14th International Conference on (pp. 148-155). IEEE.  Available online at `<https://www.ietf.org/mail-archive/web/iccrg/current/pdfB57AZSheOH.pdf>`_.

.. [Pan16] R. Pan, P. Natarajan, F. Baker, G. White, B. VerSteeg, M.S. Prabhu, C. Piglione, V. Subramanian, Internet-Draft: PIE: A lightweight control scheme to address the bufferbloat problem, April 2016.  Available online at `<https://tools.ietf.org/html/draft-ietf-aqm-pie-07>`_.

.. [Kuhn16] Kuhn, N., & Ros, D. (2016). Improving PIE's performance over high-delay paths. Available online at `<https://arxiv.org/abs/1602.00569>`_.

.. [Buf14] Bufferbloat.net.  Available online at `<http://www.bufferbloat.net/>`_.

Attributes
==========

The key attributes that the PieQueue class holds include the following: 

* ``Mode:`` PIE operating mode (BYTES or PACKETS). The default mode is PACKETS. 
* ``QueueLimit:`` The maximum number of bytes or packets the queue can hold. The default value is 25 bytes / packets.
* ``MeanPktSize:`` Mean packet size in bytes. The default value is 1000 bytes.
* ``Tupdate:`` Time period to calculate drop probability. The default value is 30 ms. 
* ``Supdate:`` Start time of the update timer. The default value is 0 ms. 
* ``DequeueThreshold:`` Minimum queue size in bytes before dequeue rate is measured. The default value is 10000 bytes. 
* ``QueueDelayReference:`` Desired queue delay. The default value is 20 ms. 
* ``MaxBurstAllowance:`` Current max burst allowance in seconds before random drop. The default value is 0.1 seconds.
* ``A:`` Value of alpha. The default value is 0.125.
* ``B:`` Value of beta. The default value is 1.25.

In addition to PIE attributes, MADPIE queue requires following attributes:

* ``MADPIE:`` MADPIE is a boolean attribute to enable MADPIE. The default value is set to false.
* ``DelayHard:`` Hard queue delay; MADPIE starts deterministic packet drops after this. The default value is 30 ms.

Examples
========

The example for PIE and MADPIE is `pie-example.cc` located in ``src/traffic-control/examples``.  To run the file (the first invocation below shows the available
command-line options):

:: 
   
   $ ./waf --run "pie-example --PrintHelp"
   $ ./waf --run "pie-example --writePcap=1" 

The expected output from the previous commands are 10 .pcap files.

Validation
**********

The PIE model is tested using :cpp:class:`PieQueueDiscTestSuite` class defined in `src/traffic-control/test/pie-queue-test-suite.cc`. The suite includes 2 test cases:

* Test 1: The first test checks the enqueue/dequeue with no drops and makes sure that PIE attributes can be set correctly.
* Test 2: The second test checks the enqueue/dequeue with drops according to PIE algorithm

The test suite can be run using the following commands: 

::

  $ ./waf configure --enable-examples --enable-tests
  $ ./waf build
  $ ./test.py -s pie-queue-disc

or  

::

  $ NS_LOG="PieQueueDisc" ./waf --run "test-runner --suite=pie-queue-disc"
