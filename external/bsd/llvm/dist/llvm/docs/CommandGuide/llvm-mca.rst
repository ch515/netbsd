llvm-mca - LLVM Machine Code Analyzer
=====================================

SYNOPSIS
--------

:program:`llvm-mca` [*options*] [input]

DESCRIPTION
-----------

:program:`llvm-mca` is a performance analysis tool that uses information
available in LLVM (e.g. scheduling models) to statically measure the performance
of machine code in a specific CPU.

Performance is measured in terms of throughput as well as processor resource
consumption. The tool currently works for processors with an out-of-order
backend, for which there is a scheduling model available in LLVM.

The main goal of this tool is not just to predict the performance of the code
when run on the target, but also help with diagnosing potential performance
issues.

Given an assembly code sequence, llvm-mca estimates the IPC (Instructions Per
Cycle), as well as hardware resource pressure. The analysis and reporting style
were inspired by the IACA tool from Intel.

:program:`llvm-mca` allows the usage of special code comments to mark regions of
the assembly code to be analyzed.  A comment starting with substring
``LLVM-MCA-BEGIN`` marks the beginning of a code region. A comment starting with
substring ``LLVM-MCA-END`` marks the end of a code region.  For example:

.. code-block:: none

  # LLVM-MCA-BEGIN My Code Region
    ...
  # LLVM-MCA-END

Multiple regions can be specified provided that they do not overlap.  A code
region can have an optional description. If no user-defined region is specified,
then :program:`llvm-mca` assumes a default region which contains every
instruction in the input file.  Every region is analyzed in isolation, and the
final performance report is the union of all the reports generated for every
code region.

Inline assembly directives may be used from source code to annotate the
assembly text:

.. code-block:: c++

  int foo(int a, int b) {
    __asm volatile("# LLVM-MCA-BEGIN foo");
    a += 42;
    __asm volatile("# LLVM-MCA-END");
    a *= b;
    return a;
  }

So for example, you can compile code with clang, output assembly, and pipe it
directly into llvm-mca for analysis:

.. code-block:: bash

  $ clang foo.c -O2 -target x86_64-unknown-unknown -S -o - | llvm-mca -mcpu=btver2

Or for Intel syntax:

.. code-block:: bash

  $ clang foo.c -O2 -target x86_64-unknown-unknown -mllvm -x86-asm-syntax=intel -S -o - | llvm-mca -mcpu=btver2

OPTIONS
-------

If ``input`` is "``-``" or omitted, :program:`llvm-mca` reads from standard
input. Otherwise, it will read from the specified filename.

If the :option:`-o` option is omitted, then :program:`llvm-mca` will send its output
to standard output if the input is from standard input.  If the :option:`-o`
option specifies "``-``", then the output will also be sent to standard output.


.. option:: -help

 Print a summary of command line options.

.. option:: -mtriple=<target triple>

 Specify a target triple string.

.. option:: -march=<arch>

 Specify the architecture for which to analyze the code. It defaults to the
 host default target.

.. option:: -mcpu=<cpuname>

  Specify the processor for which to analyze the code.  By default, the cpu name
  is autodetected from the host.

.. option:: -output-asm-variant=<variant id>

 Specify the output assembly variant for the report generated by the tool.
 On x86, possible values are [0, 1]. A value of 0 (vic. 1) for this flag enables
 the AT&T (vic. Intel) assembly format for the code printed out by the tool in
 the analysis report.

.. option:: -dispatch=<width>

 Specify a different dispatch width for the processor. The dispatch width
 defaults to field 'IssueWidth' in the processor scheduling model.  If width is
 zero, then the default dispatch width is used.

.. option:: -register-file-size=<size>

 Specify the size of the register file. When specified, this flag limits how
 many temporary registers are available for register renaming purposes. A value
 of zero for this flag means "unlimited number of temporary registers".

.. option:: -iterations=<number of iterations>

 Specify the number of iterations to run. If this flag is set to 0, then the
 tool sets the number of iterations to a default value (i.e. 100).

.. option:: -noalias=<bool>

  If set, the tool assumes that loads and stores don't alias. This is the
  default behavior.

.. option:: -lqueue=<load queue size>

  Specify the size of the load queue in the load/store unit emulated by the tool.
  By default, the tool assumes an unbound number of entries in the load queue.
  A value of zero for this flag is ignored, and the default load queue size is
  used instead.

.. option:: -squeue=<store queue size>

  Specify the size of the store queue in the load/store unit emulated by the
  tool. By default, the tool assumes an unbound number of entries in the store
  queue. A value of zero for this flag is ignored, and the default store queue
  size is used instead.

.. option:: -timeline

  Enable the timeline view.

.. option:: -timeline-max-iterations=<iterations>

  Limit the number of iterations to print in the timeline view. By default, the
  timeline view prints information for up to 10 iterations.

.. option:: -timeline-max-cycles=<cycles>

  Limit the number of cycles in the timeline view. By default, the number of
  cycles is set to 80.

.. option:: -resource-pressure

  Enable the resource pressure view. This is enabled by default.

.. option:: -register-file-stats

  Enable register file usage statistics.

.. option:: -dispatch-stats

  Enable extra dispatch statistics. This view collects and analyzes instruction
  dispatch events, as well as static/dynamic dispatch stall events. This view
  is disabled by default.

.. option:: -scheduler-stats

  Enable extra scheduler statistics. This view collects and analyzes instruction
  issue events. This view is disabled by default.

.. option:: -retire-stats

  Enable extra retire control unit statistics. This view is disabled by default.

.. option:: -instruction-info

  Enable the instruction info view. This is enabled by default.

.. option:: -all-stats

  Print all hardware statistics. This enables extra statistics related to the
  dispatch logic, the hardware schedulers, the register file(s), and the retire
  control unit. This option is disabled by default.

.. option:: -all-views

  Enable all the view.

.. option:: -instruction-tables

  Prints resource pressure information based on the static information
  available from the processor model. This differs from the resource pressure
  view because it doesn't require that the code is simulated. It instead prints
  the theoretical uniform distribution of resource pressure for every
  instruction in sequence.

EXIT STATUS
-----------

:program:`llvm-mca` returns 0 on success. Otherwise, an error message is printed
to standard error, and the tool returns 1.

INTERNALS
---------
Why MCA?  For many analysis scenarios :program:`llvm-mca` (MCA) should work
just fine out of the box; however, the tools MCA provides allows for the
curious to create their own pipelines, and explore the finer details of
instruction processing.

MCA is designed to be a flexible framework allowing users to easily create
custom instruction pipeline simulations.  The following section describes the
primary components necessary for creating a pipeline, namely the classes
``Pipeline``, ``Stage``, ``HardwareUnit``, and ``View``.

In most cases, creating a custom pipeline is not necessary, and using the
default ``mca::Context::createDefaultPipeline`` will work just fine.  Instead,
a user will probably find it easier, and faster, to implement a custom
``View``, allowing them to specifically handle the processing and presenting of
data.  Views are discussed towards the end of this document, as it can be
helpful to first understand how the rest of MCA is structured and where the
views sit in the bigger picture.

Primary Components of MCA
^^^^^^^^^^^^^^^^^^^^^^^^^
A pipeline is a collection of stages.  Stages are the real workhorse of
MCA, since all of the instruction processing occurs within them.  A stage
operates on instructions (``InstRef``) and utilizes the simulated hardware
units (``HardwareUnit``).  We draw a strong distinction between a ``Stage`` and
a ``HardwareUnit``.  Stages make use of HardwareUnits to accomplish their
primary action, which is defined in ``mca::Stage::execute``.  HardwareUnits
maintain state and act as a mechanism for inter-stage communication.  For
instance, both ``DispatchStage`` and ``RetireStage`` stages make use of the
simulated ``RegisterFile`` hardware unit for updating the state of particular
registers.

The pipeline's role is to simply execute the stages in order.  During
execution, a stage can return ``false`` from ``mca::Stage:execute``, indicating
to the pipeline that no more executions are to continue for the current cycle.
This mechanism allows for a stage to short-circuit the rest of execution for
any cycle.

Views
^^^^^
Of course simulating a pipeline is great, but it's not very useful if a user
cannot extract data from it!  This is where views come into play. The goal of a
``View`` is to collect events from the pipeline's stages.  The view can
analyze and present this collected information in a more comprehensible manner.

If the default views provided by MCA are not sufficient, then a user might
consider implementing a custom data collection and presentation mechanism (a
``View``).  Views receive callback notifications from the pipeline simulation,
specifically from the stages.  To accomplish this communication, stages contain
a list of listeners.  A view is a listener (``HWEventListener``) and can be
added to a single stage's list of listeners, or to all stages lists, by
expressing interest to be notified when particular hardware events occur (e.g.,
a hardware stall).

Notifications are generated within the stages.  When an event occurs, the stage
will iterate through its list of listeners (presumably a View) and will send
an event object describing the situation to the Listener.

What Data does a View Collect?
""""""""""""""""""""""""""""""
The two primary event types sent to views are instruction events
(``HWInstructionEvent``) and stall events (``HWStallEvent``).  The former
describes the state of an instruction (e.g., Ready, Dispatched, Executed,
etc.).  The latter describes a stall hazard (e.g., load stall, store stall,
scheduler stall, etc.).

In addition to the instruction and stall events.  A listener can also
subscribe to cycle events (``onCycleStart``, ``onCycleEnd``).  These events
occur before and after a simulated clock cycle is executed, respectively.

Listeners can also be notified of various resource states within the stages
of a pipeline, such as resource availability, reservation, and reclaim:
(``onResourceAvailability``, ``onReservedBuffers``, ``onReleasedBuffers``).

Creating a Custom View
""""""""""""""""""""""
To create a  custom view, the user must first inherit from the ``View`` class
and then decide which events are of interest.  The ``HWEventListener`` class
declares the callback functions for the particular event types.  A custom view
should define the callbacks for the events of interest.

Next, the view should define a ``mca::View::printView`` method.  This is called
once the pipeline has completed executing all of the desired cycles.  A
user can choose to perform analysis in the aforementioned routine, or do the
analysis incrementally as the event callbacks are triggered.  All presentation
of the data should be performed in ``printView``.

With a view created, the next step is to tell the pipeline's stages about it.
The simplest way to accomplish this is to create a ``PipelinePrinter`` object
and call ``mca::PipelinePrinter::addView``.  We have not discussed the
PipelinePrinter before, but it is simply a helper class containing a collection
of views and a pointer to the pipeline instance.  When ``addView`` is called,
the printer will take the liberty of registering the view as a listener to all
of the stages in the pipeline.  The printer provides a ``printReport`` routine
that iterates across all views and calls each view's ``printView`` method.
