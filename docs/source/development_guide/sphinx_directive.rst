.. _sphinx_directive:

IPython Sphinx Directive
========================

.. attention::
    This is copied verbatim from the old IPython wiki and is currently under development. Much of the information in this part of the development guide is out of date.

The ipython directive is a stateful ipython shell for embedding in
sphinx documents.  It knows about standard ipython prompts, and
extracts the input and output lines.  These prompts will be renumbered
starting at ``1``.  The inputs will be fed to an embedded ipython
interpreter and the outputs from that interpreter will be inserted as
well.  For example, code blocks like the following::

  .. code:: python3

     In [136]: x = 2

     In [137]: x**3
     Out[137]: 8

will be rendered as

.. code:: python3

   In [136]: x = 2

   In [137]: x**3
   Out[137]: 8

.. note::
    
   This tutorial should be read side-by-side with the Sphinx source
   for this document because otherwise you will see only the rendered 
   output and not the code that generated it.  Excepting the example 
   above, we will not in general be showing the literal ReST in this 
   document that generates the rendered output.
   

The state from previous sessions is stored, and standard error is
trapped. At doc build time, ipython's output and std err will be
inserted, and prompts will be renumbered. So the prompt below should
be renumbered in the rendered docs, and pick up where the block above
left off.

.. code:: python3

  In [138]: z = x*3   # x is recalled from previous block

  In [139]: z
  Out[139]: 6

  In [140]: print z
  --------> print(z)
  6

  In [141]: q = z[)   # this is a syntax error -- we trap ipy exceptions
  ------------------------------------------------------------
     File "<ipython console>", line 1
       q = z[)   # this is a syntax error -- we trap ipy exceptions
       ^
  SyntaxError: invalid syntax


The embedded interpreter supports some limited markup.  For example,
you can put comments in your ipython sessions, which are reported
verbatim.  There are some handy "pseudo-decorators" that let you
doctest the output.  The inputs are fed to an embedded ipython
session and the outputs from the ipython session are inserted into
your doc.  If the output in your doc and in the ipython session don't
match on a doctest assertion, an error will be


.. code:: python3

   In [1]: x = 'hello world'

   # this will raise an error if the ipython output is different
   @doctest
   In [2]: x.upper()
   Out[2]: 'HELLO WORLD'

   # some readline features cannot be supported, so we allow
   # "verbatim" blocks, which are dumped in verbatim except prompts
   # are continuously numbered
   @verbatim
   In [3]: x.st<TAB>
   x.startswith  x.strip


Multi-line input is supported. 

.. code:: python3

   In [130]: url = 'http://ichart.finance.yahoo.com/table.csv?s=CROX\
      .....: &d=9&e=22&f=2009&g=d&a=1&br=8&c=2006&ignore=.csv'

   In [131]: print url.split('&')
   --------> print(url.split('&'))
   ['http://ichart.finance.yahoo.com/table.csv?s=CROX', 'd=9', 'e=22',

You can do doctesting on multi-line output as well.  Just be careful
when using non-deterministic inputs like random numbers in the ipython
directive, because your inputs are ruin through a live interpreter, so
if you are doctesting random output you will get an error.  Here we
"seed" the random number generator for deterministic output, and we
suppress the seed line so it doesn't show up in the rendered output

.. code:: python3

   In [133]: import numpy.random

   @suppress
   In [134]: numpy.random.seed(2358)

   @doctest
   In [135]: numpy.random.rand(10,2)
   Out[135]:
   array([[ 0.64524308,  0.59943846],
    [ 0.47102322,  0.8715456 ],
    [ 0.29370834,  0.74776844],
    [ 0.99539577,  0.1313423 ],
    [ 0.16250302,  0.21103583],
    [ 0.81626524,  0.1312433 ],
    [ 0.67338089,  0.72302393],
    [ 0.7566368 ,  0.07033696],
    [ 0.22591016,  0.77731835],
    [ 0.0072729 ,  0.34273127]])


Another demonstration of multi-line input and output

.. code:: python3

   In [106]: print x
   --------> print(x)
   jdh

   In [109]: for i in range(10):
      .....:     print i
      .....:
      .....:
   0
   1
   2
   3
   4
   5
   6
   7
   8
   9


Most of the "pseudo-decorators" can be used an options to ipython
mode.  For example, to setup matplotlib pylab but suppress the output,
you can do.  When using the matplotlib ``use`` directive, it should
occur before any import of pylab.  This will not show up in the
rendered docs, but the commands will be executed in the embedded
interpreter and subsequent line numbers will be incremented to reflect
the inputs::


  .. code:: python3

     In [144]: from pylab import *

     In [145]: ion()

.. code:: python3

   In [144]: from pylab import *

   In [145]: ion()

Likewise, you can set ``:doctest:`` or ``:verbatim:`` to apply these
settings to the entire block.  For example,

.. code:: python3

   In [9]: cd mpl/examples/
   /home/jdhunter/mpl/examples

   In [10]: pwd
   Out[10]: '/home/jdhunter/mpl/examples'


   In [14]: cd mpl/examples/<TAB>
   mpl/examples/animation/        mpl/examples/misc/
   mpl/examples/api/              mpl/examples/mplot3d/
   mpl/examples/axes_grid/        mpl/examples/pylab_examples/
   mpl/examples/event_handling/   mpl/examples/widgets

   In [14]: cd mpl/examples/widgets/
   /home/msierig/mpl/examples/widgets

   In [15]: !wc *
       2    12    77 README.txt
      40    97   884 buttons.py
      26    90   712 check_buttons.py
      19    52   416 cursor.py
     180   404  4882 menu.py
      16    45   337 multicursor.py
      36   106   916 radio_buttons.py
      48   226  2082 rectangle_selector.py
      43   118  1063 slider_demo.py
      40   124  1088 span_selector.py
     450  1274 12457 total

You can create one or more pyplot plots and insert them with the
``@savefig`` decorator.

.. code:: python3

   @savefig plot_simple.png width=4in
   In [151]: plot([1,2,3]);

   # use a semicolon to suppress the output
   @savefig hist_simple.png width=4in
   In [151]: hist(np.random.randn(10000), 100);

In a subsequent session, we can update the current figure with some
text, and then resave 

.. code:: python3


   In [151]: ylabel('number')

   In [152]: title('normal distribution')

   @savefig hist_with_text.png width=4in
   In [153]: grid(True)

You can also have function definitions included in the source.

.. code:: python3

   In [3]: def square(x):   
      ...:     """
      ...:     An overcomplicated square function as an example.
      ...:     """
      ...:     if x < 0:
      ...:         x = abs(x)
      ...:     y = x * x
      ...:     return y
      ...: 

Then call it from a subsequent section.

.. code:: python3

   In [4]: square(3)
   Out [4]: 9

   In [5]: square(-2)
   Out [5]: 4


Writing Pure Python Code
------------------------

Pure python code is supported by the optional argument `python`. In this pure
python syntax you do not include the output from the python interpreter. The 
following markup::

   .. code:: python
      
      foo = 'bar'
      print foo
      foo = 2
      foo**2

Renders as

.. code:: python

   foo = 'bar'
   print foo
   foo = 2
   foo**2

We can even plot from python, using the savefig decorator, as well as, suppress
output with a semicolon

.. code:: python

   @savefig plot_simple_python.png width=4in
   plot([1,2,3]);

Similarly, std err is inserted

.. code:: python

   foo = 'bar'
   foo[)

Comments are handled and state is preserved

.. code:: python

   # comments are handled
   print foo

If you don't see the next code block then the options work.

.. code:: python

   ioff()
   ion()

Multi-line input is handled.

.. code:: python

   line = 'Multi\
           line &\
           support &\
           works'
   print line.split('&')

Functions definitions are correctly parsed

.. code:: python

   def square(x):
       """
       An overcomplicated square function as an example.
       """
       if x < 0:
           x = abs(x)
       y = x * x
       return y

And persist across sessions

.. code:: python

   print square(3)
   print square(-2)

Pretty much anything you can do with the ipython code, you can do with
with a simple python script. Obviously, though it doesn't make sense 
to use the doctest option.

Pseudo-Decorators
=================

Here are the supported decorators, and any optional arguments they
take.  Some of the decorators can be used as options to the entire
block (eg ``verbatim`` and ``suppress``), and some only apply to the
line just below them (eg ``savefig``).

@suppress

    execute the ipython input block, but suppress the input and output
    block from the rendered output.  Also, can be applied to the entire
    ``..ipython`` block as a directive option with ``:suppress:``.

@verbatim

    insert the input and output block in verbatim, but auto-increment
    the line numbers. Internally, the interpreter will be fed an empty
    string, so it is a no-op that keeps line numbering consistent.
    Also, can be applied to the entire ``..ipython`` block as a
    directive option with ``:verbatim:``.

@savefig OUTFILE [IMAGE_OPTIONS]

    save the figure to the static directory and insert it into the
    document, possibly binding it into a minipage and/or putting
    code/figure label/references to associate the code and the
    figure. Takes args to pass to the image directive (*scale*,
    *width*, etc can be kwargs); see `image options
    <http://docutils.sourceforge.net/docs/ref/rst/directives.html#image>`_
    for details.

@doctest

    Compare the pasted in output in the ipython block with the output
    generated at doc build time, and raise errors if they don????????t
    match. Also, can be applied to the entire ``..ipython`` block as a
    directive option with ``:doctest:``.

Configuration Options
=====================

ipython_savefig_dir

    The directory in which to save the figures. This is relative to the
    Sphinx source directory. The default is `html_static_path`.

ipython_rgxin

    The compiled regular expression to denote the start of IPython input 
    lines. The default is re.compile('In \[(\d+)\]:\s?(.*)\s*'). You 
    shouldn't need to change this.

ipython_rgxout

    The compiled regular expression to denote the start of IPython output 
    lines. The default is re.compile('Out\[(\d+)\]:\s?(.*)\s*'). You 
    shouldn't need to change this.


ipython_promptin

    The string to represent the IPython input prompt in the generated ReST. 
    The default is 'In [%d]:'. This expects that the line numbers are used
    in the prompt.

ipython_promptout

    The string to represent the IPython prompt in the generated ReST. The
    default is 'Out [%d]:'. This expects that the line numbers are used
    in the prompt.
