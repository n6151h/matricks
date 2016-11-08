MATRICKS: Manipulate Datasets as Matrices
=========================================

Class for importing and querying expression dataasets organized as a column-
and row-annotated  matrix.

Expression datasets contain the numeric results of one or more samples
derived from microarray assays.   Common to each of the assays is the
specific platform (microarray).   The dataset can be regarded as a table
with rows and columns.  Each column represents a single assay, and each row 
contains the assay results for a specific probe on the assay platform.  Thus,
the values in any given row are those obtained from the same probe location
on the platform.  These are referred to as `expression profiles`.

A dataset can be regarded as a table, such as this one (excerpted from
the `Goodell dataset`_):

| probe_id | HSC 1 | HSC 2 | NK 1  | NK 2  | 
| -------- | ----- | ----- | ----- | ----- |
| 45283    | 10.14 |  9.31 |   8.9 |  8.78 |
| 45284    | 12.52 | 12.63 | 12.55 | 11.96 |
| 45285    |  6.78 |  6.91 |  7.83 |  7.86 |
| 45286    |  5.58 |  5.06 |  6.69 |  6.64 |
| 45287    |  7.85 |  8.13 |  8.47 |  8.56 |
| 45288    |  8.12 |  7.17 |  8.71 |  8.08 |
| 45289    |  6.82 |  6.15 |  5.87 |  5.32 |
| 45290    | 10.55 | 10.39 |  10.7 |  9.93 |


.. _Goodell dataset: http://www.bcm.edu/db/db_fac-goodell.html

Expression datasets, with rare exception, are stored in text (i.e. flat) files
that have the following format:

* two or more rows of data, delimited by ASCII newline (\\x0a) characters.
  (Strictly speaking, there needen't be any data at all, but what's the point of that?)

* each line or row consists of two or more columns of data, delimited by ASCII TAB (\\x09) characters.

* the first column contains the key or `probe ID`, assumed to be alpha-numeric, or for the probe.

* the first row consists of labels identifying the probe ID and sample columns.  This, too, is assumed
  to be alpha-numeric.

* the second through last rows contain expression values and, aside from the first column, which
  contains the probe ID, are assumed to be floating point numbers.  In microarray parlance, 
  each row is  typically referred to as an `expression profile`.

Some datasets may differ from this format.  For instance, there may be no (first) row of labels,
or the data may be of some format other than floating point.  Provision is made for handling these
arguably special cases.  However, the default settings for instantiating `Matricks` classes
makes the foregoing assumptions about the contents of raw source data.  It is further assumed that
the source dataset is encoded in ASCII strings, requiring the conversion of all numeric data
to ``float`` type objects.

`Matricks` selection operations generally return `Matricks` objects.   These can be iterated,
row-wise, much like lists or tuples, to access individual expression profiles, the contents of which
can be retrieved using list / tuple semantics.

Instantiating a Matricks
------------------------

If no source dataset is supplied at instantiation, a `Matricks` is empty or null.  This isn't terribly useful
and operations on it will almost always return ``None`` or raise a `MatricksError` exception.

```python

 >>> # Create an empty Matricks instance.
 >>> a0 = Matricks()
 >>> print a0
    [[ ... ]]
```

A `Matricks` can be loaded with data either by providing a source at instantiation, or by using the
``load`` method.  (Internally, the instantiation approach just calls ``load``.)  The source can be
a string, a file (or file-like) object, a list or tuple, or another `Matricks` instance.  

We'll use a small, representative dataset that is nothing more than a list of lists.  

```python

 >>> test_raw_data = [\
 ['probe_id','ABC(12)','ABC(13)','DEFC0N(1)','CDDB4(5)','Jovi(1)','F774Lin(3)'],
 ... ['ELMT_3401602','7.14727114229','1.682159','6.6022379846','6.6318799406','6.63021747852','6.57620493019'],
 ... ['ELMT_3401603','6.6469632681','1.682159','6.63635120703','6.70026291599','6.67341553263','6.66361340118'],
 ... ['ELMT_3401605','9.33488740366','1.682159','9.7365656865','8.88887581915','8.70271863949','9.39432724993'],
 ... ['ELMT_3401607','6.65137398038','1.682159','6.75639196465','6.70203184527','7.05207191931','6.96818993978'],
 ... ['ELMT_3401612','6.58374160839','1.682159','7.05322172216','6.62893626542','6.51635952774','6.66963293585'],
 ... ['ELMT_3401614','6.59679034883','1.682159','6.65934616753','6.54032931162','6.5067348291','6.53686489577'],
 ... ['ELMT_3401619','6.66351268706','1.682159','6.67986657646','6.57837221187','6.78383553317','7.26045576436'],
 ... ['ELMT_3401623','6.65611759304','1.682159','6.80554955104','6.64764879202','6.6403692878','6.67254761381'],
 ... ['ELMT_3401625','10.6488388784','1.682159','10.3501936853','9.26241934526','9.84545402073','10.0755468901'],
 ... ['ELMT_3401626','7.5310613409','1.682159','8.10062767869','9.24637465474','11.2541801046','7.11119485886'],
 ... ['ELMT_3401628','6.55860431817','1.682159','6.63538974978','6.66347280086','6.68541200426','6.53825496578'],
 ... ['ELMT_3401632','6.97318937509','1.682159','6.59048802252','6.68431036403','6.67796164216','7.47303945599'],
 ... ['ELMT_3401633','7.20203718782','1.682159','7.33123060039','6.93376501527','7.40407740412','8.28373011066'],
 ... ['ELMT_3401636','11.4847211865','1.682159','11.7592497692','10.8845553078','10.7344737293','12.3247525578'],
 ... ['ELMT_3401637','9.15673736606','1.682159','9.93949907204','8.84061428541','9.69594817225','10.7308783293'],
 ... ['ELMT_3401638','0','1.682159','0','13.4536343874','13.5199773001','13.4779333646'],
 ... ['ELMT_3401639','7.23211276845','1.682159','6.95198458669','6.96898611023','6.68270691586','6.69342317943'],
 ... ['Elmt_3401639','7.23211276845','1.682159','6.95198458669','6.96898611023','6.68270691586','6.69342317943'],
 ... ['ELMT_3401644','6.66459889061','1.682159','6.65469610536','6.59303032509','6.63139625302','6.72401222705'],
 ... ['ELMT_3401645','9.48762418312','1.682159','8.8286277291','7.66907923624','8.4171269045','6.65231345481'],
 ...  ]
```

In fact, let's re-write this to look like TSV input::

```python

 >>> test_raw_data = '''\
 probe_id\\tABC(12)\\tABC(13)\\tDEFC0N(1)\\tCDDB4(5)\\tJovi(1)\\tF774Lin(3)
 ... ELMT_3401602\\t7.14727114229\\t1.682159\\t6.6022379846\\t6.6318799406\\t6.63021747852\\t6.57620493019
 ... ELMT_3401603\\t6.6469632681\\t1.682159\\t6.63635120703\\t6.70026291599\\t6.67341553263\\t6.66361340118
 ... ELMT_3401605\\t9.33488740366\\t1.682159\\t9.7365656865\\t8.88887581915\\t8.70271863949\\t9.39432724993
 ... ELMT_3401607\\t6.65137398038\\t1.682159\\t6.75639196465\\t6.70203184527\\t7.05207191931\\t6.96818993978
 ... ELMT_3401612\\t6.58374160839\\t1.682159\\t7.05322172216\\t6.62893626542\\t6.51635952774\\t6.66963293585
 ... ELMT_3401614\\t6.59679034883\\t1.682159\\t6.65934616753\\t6.54032931162\\t6.5067348291\\t6.53686489577
 ... ELMT_3401619\\t6.66351268706\\t1.682159\\t6.67986657646\\t6.57837221187\\t6.78383553317\\t7.26045576436
 ... ELMT_3401623\\t6.65611759304\\t1.682159\\t6.80554955104\\t6.64764879202\\t6.6403692878\\t6.67254761381
 ... ELMT_3401625\\t10.6488388784\\t1.682159\\t10.3501936853\\t9.26241934526\\t9.84545402073\\t10.0755468901
 ... ELMT_3401626\\t7.5310613409\\t1.682159\\t8.10062767869\\t9.24637465474\\t11.2541801046\\t7.11119485886
 ... ELMT_3401628\\t6.55860431817\\t1.682159\\t6.63538974978\\t6.66347280086\\t6.68541200426\\t6.53825496578
 ... ELMT_3401632\\t6.97318937509\\t1.682159\\t6.59048802252\\t6.68431036403\\t6.67796164216\\t7.47303945599
 ... ELMT_3401633\\t7.20203718782\\t1.682159\\t7.33123060039\\t6.93376501527\\t7.40407740412\\t8.28373011066
 ... ELMT_3401636\\t11.4847211865\\t1.682159\\t11.7592497692\\t10.8845553078\\t10.7344737293\\t12.3247525578
 ... ELMT_3401637\\t9.15673736606\\t1.682159\\t9.93949907204\\t8.84061428541\\t9.69594817225\\t10.7308783293
 ... ELMT_3401638\\t0\\t1.682159\\t0\\t13.4536343874\\t13.5199773001\\t13.4779333646
 ... ELMT_3401639\\t7.23211276845\\t1.682159\\t6.95198458669\\t6.96898611023\\t6.68270691586\\t6.69342317943
 ... Elmt_3401639\\t7.23211276845\\t1.682159\\t6.95198458669\\t6.96898611023\\t6.68270691586\\t6.69342317943
 ... ELMT_3401644\\t6.66459889061\\t1.682159\\t6.65469610536\\t6.59303032509\\t6.63139625302\\t6.72401222705
 ... ELMT_3401645\\t9.48762418312\\t1.682159\\t8.8286277291\\t7.66907923624\\t8.4171269045\\t6.65231345481'''

```

We then create the instance that will hold this dataset::

```python

 >>> a_pre1 = Matricks(test_raw_data, cvt=float)

```

Notice that we specified ``cvt=float``. Elements that are in rows before the first row and to the right of
the first column will be converted to type ``float`` using the float constructor.  We can use more elaborate
functions to do this conversion, but this will do for now.  Also note that because the file is otherwise in
TSV format, we didn't have to specify any additional arguments.   If it were comma-separated (CSV rather than
TSV, instance creation would look like this:

```python

 >>> test_raw_data_csv = '''\
 probe_id,ABC(12),ABC(13),DEFC0N(1),CDDB4(5),Jovi(1),F774Lin(3)
 ... ELMT_3401602,7.14727114229,1.682159,6.6022379846,6.6318799406,6.63021747852,6.57620493019
 ... ELMT_3401603,6.6469632681,1.682159,6.63635120703,6.70026291599,6.67341553263,6.66361340118
 ... ELMT_3401605,9.33488740366,1.682159,9.7365656865,8.88887581915,8.70271863949,9.39432724993
 ... ELMT_3401607,6.65137398038,1.682159,6.75639196465,6.70203184527,7.05207191931,6.96818993978
 ... ELMT_3401612,6.58374160839,1.682159,7.05322172216,6.62893626542,6.51635952774,6.66963293585
 ... ELMT_3401614,6.59679034883,1.682159,6.65934616753,6.54032931162,6.5067348291,6.53686489577
 ... ELMT_3401619,6.66351268706,1.682159,6.67986657646,6.57837221187,6.78383553317,7.26045576436
 ... ELMT_3401623,6.65611759304,1.682159,6.80554955104,6.64764879202,6.6403692878,6.67254761381
 ... ELMT_3401625,10.6488388784,1.682159,10.3501936853,9.26241934526,9.84545402073,10.0755468901
 ... ELMT_3401626,7.5310613409,1.682159,8.10062767869,9.24637465474,11.2541801046,7.11119485886
 ... ELMT_3401628,6.55860431817,1.682159,6.63538974978,6.66347280086,6.68541200426,6.53825496578
 ... ELMT_3401632,6.97318937509,1.682159,6.59048802252,6.68431036403,6.67796164216,7.47303945599
 ... ELMT_3401633,7.20203718782,1.682159,7.33123060039,6.93376501527,7.40407740412,8.28373011066
 ... ELMT_3401636,11.4847211865,1.682159,11.7592497692,10.8845553078,10.7344737293,12.3247525578
 ... ELMT_3401637,9.15673736606,1.682159,9.93949907204,8.84061428541,9.69594817225,10.7308783293
 ... ELMT_3401638,0,1.682159,0,13.4536343874,13.5199773001,13.4779333646
 ... ELMT_3401639,7.23211276845,1.682159,6.95198458669,6.96898611023,6.68270691586,6.69342317943
 ... Elmt_3401639,7.23211276845,1.682159,6.95198458669,6.96898611023,6.68270691586,6.69342317943
 ... ELMT_3401644,6.66459889061,1.682159,6.65469610536,6.59303032509,6.63139625302,6.72401222705
 ... ELMT_3401645,9.48762418312,1.682159,8.8286277291,7.66907923624,8.4171269045,6.65231345481'''
 
 >>> a_pre1_csv = Matricks(test_raw_data_csv, fsep=',', cvt=float)

```

We can see an abridged depiction of these by using ``print``::

```python

 >>> print a_pre1
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401602' 7.14727114229 1.682159 ... 6.6318799406 6.63021747852 6.57620493019],
      ['ELMT_3401603' 6.6469632681 1.682159 ... 6.70026291599 6.67341553263 6.66361340118],
      ['ELMT_3401605' 9.33488740366 1.682159 ... 8.88887581915 8.70271863949 9.39432724993],
    ...  
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['ELMT_3401644' 6.66459889061 1.682159 ... 6.59303032509 6.63139625302 6.72401222705],
      ['ELMT_3401645' 9.48762418312 1.682159 ... 7.66907923624 8.4171269045 6.65231345481]]

 >>> print a_pre1_csv
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401602' 7.14727114229 1.682159 ... 6.6318799406 6.63021747852 6.57620493019],
      ['ELMT_3401603' 6.6469632681 1.682159 ... 6.70026291599 6.67341553263 6.66361340118],
      ['ELMT_3401605' 9.33488740366 1.682159 ... 8.88887581915 8.70271863949 9.39432724993],
    ...  
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['ELMT_3401644' 6.66459889061 1.682159 ... 6.59303032509 6.63139625302 6.72401222705],
      ['ELMT_3401645' 9.48762418312 1.682159 ... 7.66907923624 8.4171269045 6.65231345481]]

```

Call ``getLabels`` to see the columns labels.

```python

 >>> print a_pre1.getLabels()
    ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']

We can specify which labels we want, either by their index::

 >>> print a_pre1.getLabels(3, 6, 0)
   ['DEFC0N(1)', 'F774Lin(3)', 'probe_id']

or we can use a string or regular expression::

 >>> print a_pre1.getLabels('ABC(12)', 'CDDB4.*', '.*\(1)', re=True)
    ['ABC(12)', ['CDDB4(5)'], ['DEFC0N(1)', 'Jovi(1)']]
```

We could have also specified a string for the data source, which would have been
used to open a file or remote URL (internally, urllib is used in either case), or we
could have specified a file-like object (from the built-in ``open`` function 
or an explicit ``urllib.urlopen`` that we did ourselves prior to
instantiating a `Matricks`.  These are difficult to test using doctest, though,
so we'll leave those for another test scenario.  

The last way we can populate an instance is with another instance::

```python

 >>> a1 = Matricks(a_pre1)
 >>> print a1.getLabels()
    ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']

 >>> for r in a1: print r
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978]
    ['ELMT_3401612', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585]
    ['ELMT_3401614', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577]
    ['ELMT_3401619', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436]
    ['ELMT_3401623', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381]
    ['ELMT_3401625', 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901]
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886]
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401632', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066]
    ['ELMT_3401636', 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578]
    ['ELMT_3401637', 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293]
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646]
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['ELMT_3401644', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705]
    ['ELMT_3401645', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481]

```

Let's check to make sure they're equal::

```python

 >>> print False not in [ t[0] == t[1] for t in zip(a_pre1, a1 ) ]
    True
```

Note again that, since the ``zip`` built-in function expects iterable objects, it treats
the two `Matricks` instances as though they were.  This works in most, if not all other cases
that handle iterables.

Writing Data:  Files
--------------------

Data can be exported from a `Matricks` instance a couple of different ways.  The
`dumps` method  converts the labels and data into one (possibly very long) string
in a format that can be (re)loaded using the `load` method without any non-default options.   As such, it
essentially writes a TAB-separated version (TSV) of the data that can be written as-is to a file or 
stream.  
Here's an example of how to use the `dumps` method::

```python

 >>> a1_s = a1.dumps()
 >>> print a1_s
    probe_id	ABC(12)	ABC(13)	DEFC0N(1)	CDDB4(5)	Jovi(1)	F774Lin(3)
    ELMT_3401602	7.14727114229	1.682159	6.6022379846	6.6318799406	6.63021747852	6.57620493019
    ELMT_3401603	6.6469632681	1.682159	6.63635120703	6.70026291599	6.67341553263	6.66361340118
    ELMT_3401605	9.33488740366	1.682159	9.7365656865	8.88887581915	8.70271863949	9.39432724993
    ELMT_3401607	6.65137398038	1.682159	6.75639196465	6.70203184527	7.05207191931	6.96818993978
    ELMT_3401612	6.58374160839	1.682159	7.05322172216	6.62893626542	6.51635952774	6.66963293585
    ELMT_3401614	6.59679034883	1.682159	6.65934616753	6.54032931162	6.5067348291	6.53686489577
    ELMT_3401619	6.66351268706	1.682159	6.67986657646	6.57837221187	6.78383553317	7.26045576436
    ELMT_3401623	6.65611759304	1.682159	6.80554955104	6.64764879202	6.6403692878	6.67254761381
    ELMT_3401625	10.6488388784	1.682159	10.3501936853	9.26241934526	9.84545402073	10.0755468901
    ELMT_3401626	7.5310613409	1.682159	8.10062767869	9.24637465474	11.2541801046	7.11119485886
    ELMT_3401628	6.55860431817	1.682159	6.63538974978	6.66347280086	6.68541200426	6.53825496578
    ELMT_3401632	6.97318937509	1.682159	6.59048802252	6.68431036403	6.67796164216	7.47303945599
    ELMT_3401633	7.20203718782	1.682159	7.33123060039	6.93376501527	7.40407740412	8.28373011066
    ELMT_3401636	11.4847211865	1.682159	11.7592497692	10.8845553078	10.7344737293	12.3247525578
    ELMT_3401637	9.15673736606	1.682159	9.93949907204	8.84061428541	9.69594817225	10.7308783293
    ELMT_3401638	0.0	1.682159	0.0	13.4536343874	13.5199773001	13.4779333646
    ELMT_3401639	7.23211276845	1.682159	6.95198458669	6.96898611023	6.68270691586	6.69342317943
    Elmt_3401639	7.23211276845	1.682159	6.95198458669	6.96898611023	6.68270691586	6.69342317943
    ELMT_3401644	6.66459889061	1.682159	6.65469610536	6.59303032509	6.63139625302	6.72401222705
    ELMT_3401645	9.48762418312	1.682159	8.8286277291	7.66907923624	8.4171269045	6.65231345481
```

The contents of a `Matricks` can be written to a file using the
`dump` or `dumps` methods.   The latter creates a string-enggded version of
the instance that can, if read from a file, be ingested by `load` with no other
option changes (i.e., *fsep* and *rsep* defaults will work.)  The `dump` method
just writes the output of `dumps` to the specified file or file-like object::

```python

 >>> import os
 >>> try:
 ...     os.unlink('/tmp/matricks_doctest.tsv')  # in case left over from earlier run
 ... except:
 ...     pass
 
 >>> print a1.dump('/tmp/matricks_doctest.tsv')
     1873
```

We can read this into another `Matricks`::

```python
 >>> a1reloaded = Matricks('/tmp/matricks_doctest.tsv', cvt=float)
```

And, again, see that they're equal::

```python
 >>> print False not in [ t[0] == t[1] for t in zip(a1, a1reloaded) ]
    True
```

(Clean up after ourselves.)::

```python
 >>> os.unlink('/tmp/matricks_doctest.tsv') 
```

####Pickling

`Matricks` instances can be pickled::

```python
 >>> import cPickle
 >>> a1_save = cPickle.dumps(a1, -1)   # Use highest protocol
```

and unpickled::

```python

 >>> a1_restored = cPickle.loads(a1_save)
 >>> print type(a1_restored)
    <class '__main__.Matricks'>
 >>> print a1_restored
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401602' 7.14727114229 1.682159 ... 6.6318799406 6.63021747852 6.57620493019],
      ['ELMT_3401603' 6.6469632681 1.682159 ... 6.70026291599 6.67341553263 6.66361340118],
      ['ELMT_3401605' 9.33488740366 1.682159 ... 8.88887581915 8.70271863949 9.39432724993],
    ...  
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['ELMT_3401644' 6.66459889061 1.682159 ... 6.59303032509 6.63139625302 6.72401222705],
      ['ELMT_3401645' 9.48762418312 1.682159 ... 7.66907923624 8.4171269045 6.65231345481]]
```

Pickling and unpickling is a much faster way to store and retrieve `Matricks`
instances, and using the higher pickle protocols provides the fastest speeds since
they use binary formats, which are closer to the internal encodings.  
In fact, if a string or a file-like object is passed to the constructor (or the `load` method)
for the input source, an attempt is first made to unpickle whatever's read in.  If this fails, the
input is otherwise split and parsed.

```python

 >>> cx = cPickle.dump(a1, open('/tmp/a1.P', 'w'), -1)
 >>> cy = Matricks(open('/tmp/a1.P', 'r'))
 >>> print False not in [ t[0] == t[1] for t in zip(a1, cy ) ]
    True
 >>> os.unlink('/tmp/a1.P')
```

**Caveat**: since functions cannot be pickled, the default aggregator function
will be set to ``Matricks.arithmetic_mean`` when an instance is unpickled.  If some other
function had been specified when the instance was originally constructed, it will have to be
explicitly reset to this after unpickling.

####JSON

A convenience method -- ``json`` -- is provided that will return a simple rendition into
string (JSON) format of the labels and data contained within a `Matricks` instance.

```python

 >>> ja1 = a1.json()
 >>> print ja1
    [["probe_id", "ABC(12)", "ABC(13)", "DEFC0N(1)", "CDDB4(5)", "Jovi(1)", "F774Lin(3)"], ["ELMT_3401602", 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019], ["ELMT_3401603", 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118], ["ELMT_3401605", 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993], ["ELMT_3401607", 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978], ["ELMT_3401612", 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585], ["ELMT_3401614", 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577], ["ELMT_3401619", 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436], ["ELMT_3401623", 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381], ["ELMT_3401625", 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901], ["ELMT_3401626", 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886], ["ELMT_3401628", 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578], ["ELMT_3401632", 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599], ["ELMT_3401633", 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066], ["ELMT_3401636", 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578], ["ELMT_3401637", 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293], ["ELMT_3401638", 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646], ["ELMT_3401639", 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943], ["Elmt_3401639", 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943], ["ELMT_3401644", 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705], ["ELMT_3401645", 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481]]

```

####GCT


GCT format is also supported.  Supply a file-like object to this method and it will
write the `Matricks` instance out in GCT-compatible format.  Note that some fields are 
"guessed" at (second row, `Description` column, `Accession` column.)

Example:


```python

 >>> import sys
 >>> from cStringIO import StringIO
 >>> outf = StringIO()
 >>> a1.gct(outf)
 >>> print outf.getvalue()
    #1.2
    20	6
    NAME	Description	ABC(12)	ABC(13)	DEFC0N(1)	CDDB4(5)	Jovi(1)	F774Lin(3)
    ELMT_3401602	na	7.14727114229	1.682159	6.6022379846	6.6318799406	6.63021747852	6.57620493019
    ELMT_3401603	na	6.6469632681	1.682159	6.63635120703	6.70026291599	6.67341553263	6.66361340118
    ELMT_3401605	na	9.33488740366	1.682159	9.7365656865	8.88887581915	8.70271863949	9.39432724993
    ELMT_3401607	na	6.65137398038	1.682159	6.75639196465	6.70203184527	7.05207191931	6.96818993978
    ELMT_3401612	na	6.58374160839	1.682159	7.05322172216	6.62893626542	6.51635952774	6.66963293585
    ELMT_3401614	na	6.59679034883	1.682159	6.65934616753	6.54032931162	6.5067348291	6.53686489577
    ELMT_3401619	na	6.66351268706	1.682159	6.67986657646	6.57837221187	6.78383553317	7.26045576436
    ELMT_3401623	na	6.65611759304	1.682159	6.80554955104	6.64764879202	6.6403692878	6.67254761381
    ELMT_3401625	na	10.6488388784	1.682159	10.3501936853	9.26241934526	9.84545402073	10.0755468901
    ELMT_3401626	na	7.5310613409	1.682159	8.10062767869	9.24637465474	11.2541801046	7.11119485886
    ELMT_3401628	na	6.55860431817	1.682159	6.63538974978	6.66347280086	6.68541200426	6.53825496578
    ELMT_3401632	na	6.97318937509	1.682159	6.59048802252	6.68431036403	6.67796164216	7.47303945599
    ELMT_3401633	na	7.20203718782	1.682159	7.33123060039	6.93376501527	7.40407740412	8.28373011066
    ELMT_3401636	na	11.4847211865	1.682159	11.7592497692	10.8845553078	10.7344737293	12.3247525578
    ELMT_3401637	na	9.15673736606	1.682159	9.93949907204	8.84061428541	9.69594817225	10.7308783293
    ELMT_3401638	na	0.0	1.682159	0.0	13.4536343874	13.5199773001	13.4779333646
    ELMT_3401639	na	7.23211276845	1.682159	6.95198458669	6.96898611023	6.68270691586	6.69342317943
    Elmt_3401639	na	7.23211276845	1.682159	6.95198458669	6.96898611023	6.68270691586	6.69342317943
    ELMT_3401644	na	6.66459889061	1.682159	6.65469610536	6.59303032509	6.63139625302	6.72401222705
    ELMT_3401645	na	9.48762418312	1.682159	8.8286277291	7.66907923624	8.4171269045	6.65231345481

 >>> a1gct = Matricks(outf.getvalue())
 >>> print a1gct.labels
    ['NAME', 'Description', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']

 >>> for r in a1gct: print r
    ['ELMT_3401602', 'na', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019]
    ['ELMT_3401603', 'na', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118]
    ['ELMT_3401605', 'na', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]
    ['ELMT_3401607', 'na', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978]
    ['ELMT_3401612', 'na', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585]
    ['ELMT_3401614', 'na', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577]
    ['ELMT_3401619', 'na', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436]
    ['ELMT_3401623', 'na', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381]
    ['ELMT_3401625', 'na', 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901]
    ['ELMT_3401626', 'na', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886]
    ['ELMT_3401628', 'na', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401632', 'na', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599]
    ['ELMT_3401633', 'na', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066]
    ['ELMT_3401636', 'na', 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578]
    ['ELMT_3401637', 'na', 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293]
    ['ELMT_3401638', 'na', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646]
    ['ELMT_3401639', 'na', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 'na', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['ELMT_3401644', 'na', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705]
    ['ELMT_3401645', 'na', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481]


DataTables
----------

Support for `DataTables`_
server-side processing
is provided with the `dataTablesObject` method.  Invoking this will return
a dictionary that has ``aaData``, ``aoColumns``, ``iTotalRecords``, and ``sColumns``
elements set in a way that is readily digestible by `DataTables`.  This can be called
with ``offset=`` and ``limit=`` keyword arguments to further support server-side 
pagination through large datasets.

.. _DataTables: http://www.datatables.net/

Retrieving Matricks Contents: Column Labels 
-------------------------------------------

We've already seen that we can retrieve the column labels (alternatively called
*sample names* or *sample labels*: iteration, and the `getLabels` method.

Take another look at what `getLabels` returned, above, and you'll notice
that in the pattern example it's a list of sublists.
This happens when a regular expression
is expanded, or when a sublist of explicity strings (or indexes) has been
supplied as input to `getlabels`.  If the ``flat=`` keyword argument is
set to ``True``, the result will be `flattened` into a single, unstructured list:: 

```python
 >>> print a_pre1.getLabels('ABC(12)', 'CDDB4.*', '.*\(1)', flat=True, re=True)
   ['ABC(12)', 'CDDB4(5)', 'DEFC0N(1)', 'Jovi(1)']
```

If we want the complement of this set, we can set ``cmpl=True`` in the
keyword arguments::

```python

 >>> print a_pre1.getLabels('ABC.*', cmpl=True, re=True)
    [['probe_id', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']]
```

Retrieving Matricks Contents: Data
----------------------------------

We looked at our example dataset earlier by simply iterating through it just as we would
any list of objects. 
`Matricks` instances can be used like any other iterable type (lists, tuples, etc.) with
each iteration returning the next expression profile (row) in the data set. 

```python

 >>> print len(a1) == 20 and len(a1) == len([ x for x in a1 ])
    True
 >>> print len(a0)
    0
```

Each iteration of a `Matricks` returns a python 
`list` object by default, the elements therein being associated with
their respective samples by having the same index within the list as the sample's label in the label
list (see `getLabels` method.)   A convenience function is provided that will return a dictionary
associating the elements associated with their corresponding sample labels::

```python

 >>> for r in a1: print a1.todict(r)
    OrderedDict([('probe_id', 'ELMT_3401602'), ('ABC(12)', 7.14727114229), ('ABC(13)', 1.682159), ('DEFC0N(1)', 6.6022379846), ('CDDB4(5)', 6.6318799406), ('Jovi(1)', 6.63021747852), ('F774Lin(3)', 6.57620493019)])
    OrderedDict([('probe_id', 'ELMT_3401603'), ('ABC(12)', 6.6469632681), ('ABC(13)', 1.682159), ('DEFC0N(1)', 6.63635120703), ('CDDB4(5)', 6.70026291599), ('Jovi(1)', 6.67341553263), ('F774Lin(3)', 6.66361340118)])
    			      :

```


Matricks Attributes
-------------------
`Matrickss` have several, read-only attributes and properties::

 *length* : (int)
  number of rows of data, not including the label row(s).

 *min* : (float)
  numeric minimum of all the values in the `Matricks` instance.

 *max* : (float)
  numeric maximum of all the values in the `Matricks` instance.

 *mean* : (float)
  arithmetic mean of all the values in the `Matricks` instance.


Selecting subsets
-----------------

Suppose we are interested in just one of the samples contained in this set.  We can extract this, 
creating another, separate `Matricks` instance, using python's *slice* semantics:

```python

 >>> r1 = a1['probe_id', 'DEFC0N(1)']
 >>> print r1.getLabels()
    ['probe_id', 'DEFC0N(1)']

 >>> for r in r1: print r
    ['ELMT_3401602', 6.6022379846]
    ['ELMT_3401603', 6.63635120703]
    ['ELMT_3401605', 9.7365656865]
    ['ELMT_3401607', 6.75639196465]
    ['ELMT_3401612', 7.05322172216]
    ['ELMT_3401614', 6.65934616753]
    ['ELMT_3401619', 6.67986657646]
    ['ELMT_3401623', 6.80554955104]
    ['ELMT_3401625', 10.3501936853]
    ['ELMT_3401626', 8.10062767869]
    ['ELMT_3401628', 6.63538974978]
    ['ELMT_3401632', 6.59048802252]
    ['ELMT_3401633', 7.33123060039]
    ['ELMT_3401636', 11.7592497692]
    ['ELMT_3401637', 9.93949907204]
    ['ELMT_3401638', 0.0]
    ['ELMT_3401639', 6.95198458669]
    ['Elmt_3401639', 6.95198458669]
    ['ELMT_3401644', 6.65469610536]
    ['ELMT_3401645', 8.8286277291]
```

Note how the `Matricks` is represented: as a list of lists, with each sublist containing the 
`probe ID` column(s) followed by the selected sample column(s).   Also note that the first list 
contains the column labels. 

Two create a subset comprised of more than one sample (column) there are two methods available.
One can compose a multi-sample subset using ``union``, ``intersection``, and ``modulus``
methods (or their corresponding operators, ``|``, ``&``, and ``%``.   Bear in mind that this requires the two
`Matrickss` to be sufficiently the same to effect such a composition.  Briefly, this means they need to both
have probe ID (i.e. first) column(s) be the same, both in content and in order.  The expression columns need
not be the same at all.  Probe IDs in one instance are NOT compared with those in another, so if the order isn't the
same, the result will be a dog's breakfast.

So, suppose we wanted to add another sample to the ``r1`` subset we created, above.  We can make another subset

```python
 >>> r2 = a1['probe_id', 'ABC(12)']
```

and then combine this with the first one to  create a new `Matricks` subset that contains just those two columns
from the original `Matricks`.

```python

 >>> r12 = r1 | r2   
 >>> print r12.getLabels()
   ['probe_id', 'DEFC0N(1)', 'ABC(12)']

 >>> #print r12._data

 >>> for r in r12: print r
    ['ELMT_3401602', 6.6022379846, 7.14727114229]
    ['ELMT_3401603', 6.63635120703, 6.6469632681]
    ['ELMT_3401605', 9.7365656865, 9.33488740366]
    ['ELMT_3401607', 6.75639196465, 6.65137398038]
    ['ELMT_3401612', 7.05322172216, 6.58374160839]
    ['ELMT_3401614', 6.65934616753, 6.59679034883]
    ['ELMT_3401619', 6.67986657646, 6.66351268706]
    ['ELMT_3401623', 6.80554955104, 6.65611759304]
    ['ELMT_3401625', 10.3501936853, 10.6488388784]
    ['ELMT_3401626', 8.10062767869, 7.5310613409]
    ['ELMT_3401628', 6.63538974978, 6.55860431817]
    ['ELMT_3401632', 6.59048802252, 6.97318937509]
    ['ELMT_3401633', 7.33123060039, 7.20203718782]
    ['ELMT_3401636', 11.7592497692, 11.4847211865]
    ['ELMT_3401637', 9.93949907204, 9.15673736606]
    ['ELMT_3401638', 0.0, 0.0]
    ['ELMT_3401639', 6.95198458669, 7.23211276845]
    ['Elmt_3401639', 6.95198458669, 7.23211276845]
    ['ELMT_3401644', 6.65469610536, 6.66459889061]
    ['ELMT_3401645', 8.8286277291, 9.48762418312]

```

Of course, we don't have to create named instances to do this.  We can write a python statement that will do
it all in one step:

```python
 >>> r3 = a1['probe_id', 'ABC(12)'] | a1['probe_id', 'DEFC0N(1)'] | a1['probe_id', 'F774Lin(3)']
 >>> print r3.getLabels()
    ['probe_id', 'ABC(12)', 'DEFC0N(1)', 'F774Lin(3)']

 >>> for r in r3: print r
    ['ELMT_3401602', 7.14727114229, 6.6022379846, 6.57620493019]
    ['ELMT_3401603', 6.6469632681, 6.63635120703, 6.66361340118]
    ['ELMT_3401605', 9.33488740366, 9.7365656865, 9.39432724993]
    ['ELMT_3401607', 6.65137398038, 6.75639196465, 6.96818993978]
    ['ELMT_3401612', 6.58374160839, 7.05322172216, 6.66963293585]
    ['ELMT_3401614', 6.59679034883, 6.65934616753, 6.53686489577]
    ['ELMT_3401619', 6.66351268706, 6.67986657646, 7.26045576436]
    ['ELMT_3401623', 6.65611759304, 6.80554955104, 6.67254761381]
    ['ELMT_3401625', 10.6488388784, 10.3501936853, 10.0755468901]
    ['ELMT_3401626', 7.5310613409, 8.10062767869, 7.11119485886]
    ['ELMT_3401628', 6.55860431817, 6.63538974978, 6.53825496578]
    ['ELMT_3401632', 6.97318937509, 6.59048802252, 7.47303945599]
    ['ELMT_3401633', 7.20203718782, 7.33123060039, 8.28373011066]
    ['ELMT_3401636', 11.4847211865, 11.7592497692, 12.3247525578]
    ['ELMT_3401637', 9.15673736606, 9.93949907204, 10.7308783293]
    ['ELMT_3401638', 0.0, 0.0, 13.4779333646]
    ['ELMT_3401639', 7.23211276845, 6.95198458669, 6.69342317943]
    ['Elmt_3401639', 7.23211276845, 6.95198458669, 6.69342317943]
    ['ELMT_3401644', 6.66459889061, 6.65469610536, 6.72401222705]
    ['ELMT_3401645', 9.48762418312, 8.8286277291, 6.65231345481]
```

Note that with unions, columns are not duplicated.  If there are columns in both `Matricks` instances
with the same label, only the left-hand column will be used in the resulting instance.   Even though
you could theoretically select samples from two different parent `Matricks` instances, sample labels should be
(and, in practice, always are) unique.  Consequently, we assume samples with the same label are the same
sample and therefore need not be duplicated when composing.

Python slice attributes can be lists as well as strings or numbers.   This provides
an alternate way of construcing subsets from a larger `Matricks`.
To do this, we simply take a slice, passing the list of samples, rather than just one
sample name, as the first (or only) element of the slice.
To create a new `Matricks` that includes only the ``DEFC0N(1) and ``ABC(12)`` 
samples::

```python
 >>> r5 = a1[['probe_id', 'DEFC0N(1)', 'ABC(12)']]
 >>> print r5.getLabels()
    ['probe_id', 'DEFC0N(1)', 'ABC(12)']

 >>> for r in r5: print r
    ['ELMT_3401602', 6.6022379846, 7.14727114229]
    ['ELMT_3401603', 6.63635120703, 6.6469632681]
    ['ELMT_3401605', 9.7365656865, 9.33488740366]
    ['ELMT_3401607', 6.75639196465, 6.65137398038]
    ['ELMT_3401612', 7.05322172216, 6.58374160839]
    ['ELMT_3401614', 6.65934616753, 6.59679034883]
    ['ELMT_3401619', 6.67986657646, 6.66351268706]
    ['ELMT_3401623', 6.80554955104, 6.65611759304]
    ['ELMT_3401625', 10.3501936853, 10.6488388784]
    ['ELMT_3401626', 8.10062767869, 7.5310613409]
    ['ELMT_3401628', 6.63538974978, 6.55860431817]
    ['ELMT_3401632', 6.59048802252, 6.97318937509]
    ['ELMT_3401633', 7.33123060039, 7.20203718782]
    ['ELMT_3401636', 11.7592497692, 11.4847211865]
    ['ELMT_3401637', 9.93949907204, 9.15673736606]
    ['ELMT_3401638', 0.0, 0.0]
    ['ELMT_3401639', 6.95198458669, 7.23211276845]
    ['Elmt_3401639', 6.95198458669, 7.23211276845]
    ['ELMT_3401644', 6.65469610536, 6.66459889061]
    ['ELMT_3401645', 8.8286277291, 9.48762418312]
```

As we saw earlier, sample names can be regular expressions, allowing us
to specify a subset using a pattern.   If we had another `LSK` column (say, `ABC(13)`)
we could create another `Matricks` instance with just those two columns
thusly::

```python
 >>> r5lsk = a1.get(['probe_id', 'ABC.*'], re=True)
 >>> print r5lsk.getLabels()
    ['probe_id', 'ABC(12)', 'ABC(13)']

 >>> for r in r5lsk: print r
    ['ELMT_3401602', 7.14727114229, 1.682159]
    ['ELMT_3401603', 6.6469632681, 1.682159]
    ['ELMT_3401605', 9.33488740366, 1.682159]
    ['ELMT_3401607', 6.65137398038, 1.682159]
    ['ELMT_3401612', 6.58374160839, 1.682159]
    ['ELMT_3401614', 6.59679034883, 1.682159]
    ['ELMT_3401619', 6.66351268706, 1.682159]
    ['ELMT_3401623', 6.65611759304, 1.682159]
    ['ELMT_3401625', 10.6488388784, 1.682159]
    ['ELMT_3401626', 7.5310613409, 1.682159]
    ['ELMT_3401628', 6.55860431817, 1.682159]
    ['ELMT_3401632', 6.97318937509, 1.682159]
    ['ELMT_3401633', 7.20203718782, 1.682159]
    ['ELMT_3401636', 11.4847211865, 1.682159]
    ['ELMT_3401637', 9.15673736606, 1.682159]
    ['ELMT_3401638', 0.0, 1.682159]
    ['ELMT_3401639', 7.23211276845, 1.682159]
    ['Elmt_3401639', 7.23211276845, 1.682159]
    ['ELMT_3401644', 6.66459889061, 1.682159]
    ['ELMT_3401645', 9.48762418312, 1.682159]
```

Just as we can join two samples, you can also take their intersection::

```python
 >>> r3a = r3 & a1['probe_id', 'CDDB4(5)']  # This should result in an essentially null set.
 >>> print a1.labels
    ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']

 >>> print r3a.getLabels()
    ['probe_id']

 >>> for r in r3a: print r
    ['ELMT_3401602']
    ['ELMT_3401603']
    ['ELMT_3401605']
    ['ELMT_3401607']
    ['ELMT_3401612']
    ['ELMT_3401614']
    ['ELMT_3401619']
    ['ELMT_3401623']
    ['ELMT_3401625']
    ['ELMT_3401626']
    ['ELMT_3401628']
    ['ELMT_3401632']
    ['ELMT_3401633']
    ['ELMT_3401636']
    ['ELMT_3401637']
    ['ELMT_3401638']
    ['ELMT_3401639']
    ['Elmt_3401639']
    ['ELMT_3401644']
    ['ELMT_3401645']
```

There is also a way to take the complement of a `Matricks`.  Given a parent `Matricks` **A** and a subset **B** we can obtain 
the complement of **B** -- that is, the samples in **A** that are *not* found in **B** -- by using the ``modulus`` method or operator (%).
Using ``r12`` instance we created earlier, we can find the complementaruy subset thusly::

```python
 >>> # "modulo" of two sets:  a % b is what's left when you remove the rows of b from a.
 >>> r3b = a1 % r12
 >>> print r3b.getLabels()
    ['probe_id', 'ABC(13)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']

 >>> for r in r3b: print r
    ['ELMT_3401602', 1.682159, 6.6318799406, 6.63021747852, 6.57620493019]
    ['ELMT_3401603', 1.682159, 6.70026291599, 6.67341553263, 6.66361340118]
    ['ELMT_3401605', 1.682159, 8.88887581915, 8.70271863949, 9.39432724993]
    ['ELMT_3401607', 1.682159, 6.70203184527, 7.05207191931, 6.96818993978]
    ['ELMT_3401612', 1.682159, 6.62893626542, 6.51635952774, 6.66963293585]
    ['ELMT_3401614', 1.682159, 6.54032931162, 6.5067348291, 6.53686489577]
    ['ELMT_3401619', 1.682159, 6.57837221187, 6.78383553317, 7.26045576436]
    ['ELMT_3401623', 1.682159, 6.64764879202, 6.6403692878, 6.67254761381]
    ['ELMT_3401625', 1.682159, 9.26241934526, 9.84545402073, 10.0755468901]
    ['ELMT_3401626', 1.682159, 9.24637465474, 11.2541801046, 7.11119485886]
    ['ELMT_3401628', 1.682159, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401632', 1.682159, 6.68431036403, 6.67796164216, 7.47303945599]
    ['ELMT_3401633', 1.682159, 6.93376501527, 7.40407740412, 8.28373011066]
    ['ELMT_3401636', 1.682159, 10.8845553078, 10.7344737293, 12.3247525578]
    ['ELMT_3401637', 1.682159, 8.84061428541, 9.69594817225, 10.7308783293]
    ['ELMT_3401638', 1.682159, 13.4536343874, 13.5199773001, 13.4779333646]
    ['ELMT_3401639', 1.682159, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 1.682159, 6.96898611023, 6.68270691586, 6.69342317943]
    ['ELMT_3401644', 1.682159, 6.59303032509, 6.63139625302, 6.72401222705]
    ['ELMT_3401645', 1.682159, 7.66907923624, 8.4171269045, 6.65231345481]
```

The `pop` method works like a cross between the python ``dict``'s `pop` method
and the `matricks.modulus` method, here.  If no arguments are specified, `pop` 
returns a copy of the instance with all but the last (right-most) column.  If
a list of columns is provided, the result will have all but those columns in it.

```python
 >>> print a1
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401602' 7.14727114229 1.682159 ... 6.6318799406 6.63021747852 6.57620493019],
      ['ELMT_3401603' 6.6469632681 1.682159 ... 6.70026291599 6.67341553263 6.66361340118],
      ['ELMT_3401605' 9.33488740366 1.682159 ... 8.88887581915 8.70271863949 9.39432724993],
    ...  
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['ELMT_3401644' 6.66459889061 1.682159 ... 6.59303032509 6.63139625302 6.72401222705],
      ['ELMT_3401645' 9.48762418312 1.682159 ... 7.66907923624 8.4171269045 6.65231345481]]

 >>> a1pop = a1.pop()
 >>> print a1pop
    [['ABC(12)' 'ABC(13)' 'DEFC0N(1)' 'CDDB4(5)' 'Jovi(1)'],
      [7.14727114229 1.682159 6.6022379846 6.6318799406 6.63021747852],
      [6.6469632681 1.682159 6.63635120703 6.70026291599 6.67341553263],
      [9.33488740366 1.682159 9.7365656865 8.88887581915 8.70271863949],
    ...  
      [7.23211276845 1.682159 6.95198458669 6.96898611023 6.68270691586],
      [6.66459889061 1.682159 6.65469610536 6.59303032509 6.63139625302],
      [9.48762418312 1.682159 8.8286277291 7.66907923624 8.4171269045]]

 >>> a1pop = a1.pop(['ABC(13)', 'Jovi(1)'])
 >>> print a1pop
    [['ABC(12)' 'DEFC0N(1)' 'CDDB4(5)' 'F774Lin(3)'],
      [7.14727114229 6.6022379846 6.6318799406 6.57620493019],
      [6.6469632681 6.63635120703 6.70026291599 6.66361340118],
      [9.33488740366 9.7365656865 8.88887581915 9.39432724993],
    ...  
      [7.23211276845 6.95198458669 6.96898611023 6.69342317943],
      [6.66459889061 6.65469610536 6.59303032509 6.72401222705],
      [9.48762418312 8.8286277291 7.66907923624 6.65231345481]]
```

####Selecting Rows

One or more rows may be extracted from a `Matricks` instance by using the `extractRows` method,
passing it a string or sequence listing the key column value(s) for the row(s) of interest.  This returns
another (presumably smaller) `Matricks` instance that contains only the selected rows.

For example, to extract the row with key element ``ELMT_3401628``::

```python
 >>> swatch = a1.extractRows('ELMT_3401628')
 >>> for r in swatch: print r
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
```

To get more than one, specify more than one or specify a list::

```python
 >>> swatch = a1.extractRows('ELMT_3401628', 'ELMT_3401605')
 >>> for r in swatch: print r
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]

 >>> swatch = a1.extractRows(['ELMT_3401628', 'ELMT_3401605'])
 >>> for r in swatch: print r
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]
```

If more than one row's key column (default: column 0) matches, all matching rows will
be returned::

```python
 >>> swatch = a1.extractRows('ELMT_3401639')
 >>> for r in swatch: print r
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
```

What should be immediately obvious is that row IDs are case-insensitive.  We could just have
easily looked for ``eLmT_3401639`` and come up with the same result.  To get rows that
match without folding, use ``fold=False``::

```python
 >>> swatch = a1.extractRows('ELMT_3401639', fold=False)
 >>> for r in swatch: print r
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
```

An argument may also be a slice, but this must be constructed more or less explicitly.  This 
notation is used to specify an absolute range of rows.  For instance, suppose we wanted to 
get the rows 3 through 18, we would use::

```python
 >>> swatch = a1.extractRows(slice(3,18))
 >>> print swatch
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401607' 6.65137398038 1.682159 ... 6.70203184527 7.05207191931 6.96818993978],
      ['ELMT_3401612' 6.58374160839 1.682159 ... 6.62893626542 6.51635952774 6.66963293585],
      ['ELMT_3401614' 6.59679034883 1.682159 ... 6.54032931162 6.5067348291 6.53686489577],
    ...  
      ['ELMT_3401638' 0.0 1.682159 ... 13.4536343874 13.5199773001 13.4779333646],
      ['ELMT_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943]]
```

We even must do our own arithmetic.  If we wanted to get every other row, we can use the `step`
component of the slice, as would normally be used in any list.  To get all the odd-numbered rows
in the instance::

```python
 >>> swatch = a1.extractRows(slice(1,None,2))
 >>> print swatch
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401603' 6.6469632681 1.682159 ... 6.70026291599 6.67341553263 6.66361340118],
      ['ELMT_3401607' 6.65137398038 1.682159 ... 6.70203184527 7.05207191931 6.96818993978],
      ['ELMT_3401614' 6.59679034883 1.682159 ... 6.54032931162 6.5067348291 6.53686489577],
    ...  
      ['ELMT_3401638' 0.0 1.682159 ... 13.4536343874 13.5199773001 13.4779333646],
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['ELMT_3401645' 9.48762418312 1.682159 ... 7.66907923624 8.4171269045 6.65231345481]]
```

Rows can be further reduced by supplying `extractRows` with a *discriminator* function using the ``discrim=`` keyword argument.
This should be a function that takes a row as its sole argument and returns ``True`` or ``False``, depending on
whether the row is to be included in the result or not.  Note that this still requires a range of rows to be provided.

Example:  Suppose you want to select only those rows with that do not contain null (``None``) values::

```python
 >>> no_nulls = a1.extractRows(*range(len(a1)), discrim=lambda r: (None not in r))
 >>> for row in no_nulls: print row
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978]
    ['ELMT_3401612', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585]
    ['ELMT_3401614', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577]
    ['ELMT_3401619', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436]
    ['ELMT_3401623', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381]
    ['ELMT_3401625', 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901]
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886]
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401632', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066]
    ['ELMT_3401636', 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578]
    ['ELMT_3401637', 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293]
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646]
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['ELMT_3401644', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705]
    ['ELMT_3401645', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481]
```

Value Range Selections
----------------------

Rows can be selected by specifying lower and upper boundaries, returning
only those profiles that satisfy these criteria.    Range selection uses the same (slice) 
semantics as sample selection, above, with the first slice component specifying the
sample label.  However the result will contain all samples, not just
the one specified for the range selection.   The profiles returned will include only those 
for which the expression values for the specified sample fall within the given range. 

The range is specified using the second and third slice components to specify the low and high
bounds of the search range.  As in regular slicing, either of these may be omitted, permitting
upper bound-only or lower-bound only searches.

For the first example, suppose you wanted to create a subset of our original **a1** `Matricks`
that included only those profiles where the *ABC(12)* expression values are greater than or equal to
7.0.    Note that the upper bound has been omitted, so this search will return a `Matricks` instance
wherein all the `ABC(12)` values are >= 7.0::

```python
 >>> r4 = a1[['probe_id','ABC(12)']:7.0]
 >>> for r in r4: print r
    ['ELMT_3401602', 7.14727114229]
    ['ELMT_3401603', None]
    ['ELMT_3401605', 9.33488740366]
    ['ELMT_3401607', None]
    ['ELMT_3401612', None]
    ['ELMT_3401614', None]
    ['ELMT_3401619', None]
    ['ELMT_3401623', None]
    ['ELMT_3401625', 10.6488388784]
    ['ELMT_3401626', 7.5310613409]
    ['ELMT_3401628', None]
    ['ELMT_3401632', None]
    ['ELMT_3401633', 7.20203718782]
    ['ELMT_3401636', 11.4847211865]
    ['ELMT_3401637', 9.15673736606]
    ['ELMT_3401638', None]
    ['ELMT_3401639', 7.23211276845]
    ['Elmt_3401639', 7.23211276845]
    ['ELMT_3401644', None]
    ['ELMT_3401645', 9.48762418312]

 >>> print [ x[1] >= 7.0 for x in r4 ]
    [True, False, True, False, False, False, False, False, True, True, False, False, True, True, True, False, True, True, False, True]
```

Here are two other examples, included mainly for testing purposes.  First, one to 
demonstrate the use of both upper and lower bounds::

```python
 >>> r4a = a1[['probe_id','ABC(12)']:7.0:9]
 >>> for r in r4a: print r
    ['ELMT_3401602', 7.14727114229]
    ['ELMT_3401603', None]
    ['ELMT_3401605', None]
    ['ELMT_3401607', None]
    ['ELMT_3401612', None]
    ['ELMT_3401614', None]
    ['ELMT_3401619', None]
    ['ELMT_3401623', None]
    ['ELMT_3401625', None]
    ['ELMT_3401626', 7.5310613409]
    ['ELMT_3401628', None]
    ['ELMT_3401632', None]
    ['ELMT_3401633', 7.20203718782]
    ['ELMT_3401636', None]
    ['ELMT_3401637', None]
    ['ELMT_3401638', None]
    ['ELMT_3401639', 7.23211276845]
    ['Elmt_3401639', 7.23211276845]
    ['ELMT_3401644', None]
    ['ELMT_3401645', None]


 >>> print [ (x[1] >= 7.0 and x[1] < 9) for x in r4a ]
   [True, False, False, False, False, False, False, False, False, True, False, False, True, False, False, False, True, True, False, False]
```

Here's another to demonstrate just upper bounds::

```python
 >>> r4b = a1[['probe_id','ABC(12)']::6.6]
 >>> for r in r4b: print r
    ['ELMT_3401602', None]
    ['ELMT_3401603', None]
    ['ELMT_3401605', None]
    ['ELMT_3401607', None]
    ['ELMT_3401612', 6.58374160839]
    ['ELMT_3401614', 6.59679034883]
    ['ELMT_3401619', None]
    ['ELMT_3401623', None]
    ['ELMT_3401625', None]
    ['ELMT_3401626', None]
    ['ELMT_3401628', 6.55860431817]
    ['ELMT_3401632', None]
    ['ELMT_3401633', None]
    ['ELMT_3401636', None]
    ['ELMT_3401637', None]
    ['ELMT_3401638', 0.0]
    ['ELMT_3401639', None]
    ['Elmt_3401639', None]
    ['ELMT_3401644', None]
    ['ELMT_3401645', None]


 >>> print [ (x[1] is not None) and (x[1] < 6.6) for x in r4b ]
    [False, False, False, False, True, True, False, False, False, False, True, False, False, False, False, True, False, False, False, False]
```

Column Subset Specification
---------------------------

Whereas *range* lets us constrain the numeric values within a `Matricks`, *Column Subsets* let us
create a new `Matricks` instance by specifying a subset of columns in an existing instance.

As seen above, a range can be specified by providing explicit numeric values using *slice* semantics.
However, we are often interested in differential expression, wherein we want to know if
tha value in a given sample is higher or lower than the value for other samples in the same
profile.   `Matricks` can generate *comparand* functions on the fly from a list that
specifies the samples to be used in the comparison, as well as the aggregator function
that will be used to compute the scalar value for the comparison.   The internal method that
does this is called `graep` (pronounced like the name of the fruit from which wine is typically
made.)

Graep works by taking a list of sample names (labels) and returning an anonymous function
(created using python's ``lambda`` operator) that will take a row from the same (or *similar*)
`Matricks` instance and return the aggregated values of the expression data in that row
for the specified samples (columns).    The default aggregation function simple
takes the arithmetic mean of the values specified.   The ``agg`` keyword can
optionally be used to specify a different function.   The only requirement for
this to work properly is that it take a sequence type and return a scalar numeric
value.  (You could, in theory, have it return just about anything, or nothing, but,
as we'll see later, this is probably a bad idea.)  Incidentally, the arithmetic mean
was selected as the default aggregation function because that's what was originally
being used in the project for which this was developed.

Suppose we want to use the expression levels in
``ABC(12)``, ``CDDB4(5)`` and ``F774Lin(3)`` samples.   We pass a list with these
sample names to our ``graep`` method and get back a function that, when applied to 
a row within the same `Matricks` instance, returns a scalar for each row, derived from
the ``ABC(12)``, ``CDDB4(5)`` and ``F774Lin(3)`` samples for that row::

```python
 >>> fn1 = a1.graep(['ABC(12)', 'CDDB4(5)', 'F774Lin(3)'])
 >>> print fn1
   <function <lambda> at ...>
```

We'll use the iterator feature of the `Matricks` to apply our on-the-fly function to each of the
rows in the set.

```python
 >>> p1 = [ fn1(x) for x in a1 ]
 >>> for r in p1: print r
    6.78511867103
    6.67027986176
    9.20603015758
    6.77386525514
    6.62743693655
    6.55799485207
    6.83411355443
    6.65877133296
    9.99560170459
    7.9628769515
    6.5867773616
    7.04351306504
    7.47317743792
    11.5646763507
    9.57607666026
    8.97718925067
    6.96484068604
    6.96484068604
    6.66054714758
    7.93633895806
```

If we apply upper and or lower bounds as well, this will return a complete sample set,
but with the elements that do not meet the range criteria all set to ``None``.   The test
will be applied to *ALL* samples in the list, but return *ALL* samples in the `Matricks`.

```python
 >>> r6 = a1[['probe_id', 'ABC(12)', 'F774Lin(3)']:7.2]
 >>> print r6.getLabels()
    ['probe_id', 'ABC(12)', 'F774Lin(3)']

 >>> for r in r6:
 ...     print r
    ['ELMT_3401602', None, None]
    ['ELMT_3401603', None, None]
    ['ELMT_3401605', 9.33488740366, 9.39432724993]
    ['ELMT_3401607', None, None]
    ['ELMT_3401612', None, None]
    ['ELMT_3401614', None, None]
    ['ELMT_3401619', None, 7.26045576436]
    ['ELMT_3401623', None, None]
    ['ELMT_3401625', 10.6488388784, 10.0755468901]
    ['ELMT_3401626', 7.5310613409, None]
    ['ELMT_3401628', None, None]
    ['ELMT_3401632', None, 7.47303945599]
    ['ELMT_3401633', 7.20203718782, 8.28373011066]
    ['ELMT_3401636', 11.4847211865, 12.3247525578]
    ['ELMT_3401637', 9.15673736606, 10.7308783293]
    ['ELMT_3401638', None, 13.4779333646]
    ['ELMT_3401639', 7.23211276845, None]
    ['Elmt_3401639', 7.23211276845, None]
    ['ELMT_3401644', None, None]
    ['ELMT_3401645', 9.48762418312, None]

 >>> print [ (None not in [x[1], x[2]])  for x in r6 ]
    [False, False, True, False, False, False, False, False, True, False, False, False, True, True, True, False, False, False, False, False]
```

Joins
-----

The `join` method allows two tables to be appeneded to one another where matches are found
between the keys (first olumn) of both instances.  This differs from the concatenation
operators in that dissimilar `Matricks` instances can be joined.  For those familiar with
SQL terms, this iss strictkly an *inner join*.  If a row in the calling instance does
not have a corresponding row in the "other" instance, neither row is included in the join.
When a matching row is found, the two are concatenated to form a new, longer row in the
resulting instance.  

Likewise, the column labels are concatenated.  If the second instance has a column with 
the same label as the first, it is modified with a suffix to distinguish it from the
first instance's column. 

Suppose we take a small subste of our test data::

```python
 >>> test_raw_data_2 = [\
 ['probe_id','ABC(12)','ABC(14)'],
 ... ['ELMT_3401602','7.14727114229','1.682159'],
 ... ['ELMT_3401603','6.6469632681','1.682159'],
 ... ]
```

Create a `Matricks` instance::

```python
 >>> x1 = Matricks(test_raw_data_2, cvt=float)
 >>> print x1
    [['probe_id' 'ABC(12)' 'ABC(14)'],
      ['ELMT_3401602' 7.14727114229 1.682159],
      ['ELMT_3401603' 6.6469632681 1.682159]]
```

and join it to our larger, original test `Matricks`::

```python
 >>> j1 = a1.join(x1)
 >>> print j1.getLabels()
    ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)', 'ABC(12)', 'ABC(14)']
```

Here is what we get::

```python
 >>> for r in j1: print r
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019, 7.14727114229, 1.682159]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118, 6.6469632681, 1.682159]
```

We can also do a sort of outer join.  All this really does is fill in "blank" entries for rows where
there is no match.  Activate this by setting the ``outer=`` option to this filler value.  Can be
anything but ``False``::

```python
 >>> test_raw_data_3 = [
 ... ['probe_id','gene','chromo'],
 ... ['ELMT_999999','xyz','12'],
 ... ]

 >>> md = Matricks(test_raw_data_3, skipcols=3)
 >>> jd = a1.join(md, outer=None)
 >>> print jd.labels
    ['probe_id', 'gene', 'chromo', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']

 >>> for r in jd: print r
    ['ELMT_3401602', None, None, 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019]
    ['ELMT_3401603', None, None, 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118]
    ['ELMT_3401605', None, None, 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]
    ['ELMT_3401607', None, None, 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978]
    ['ELMT_3401612', None, None, 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585]
    ['ELMT_3401614', None, None, 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577]
    ['ELMT_3401619', None, None, 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436]
    ['ELMT_3401623', None, None, 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381]
    ['ELMT_3401625', None, None, 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901]
    ['ELMT_3401626', None, None, 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886]
    ['ELMT_3401628', None, None, 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401632', None, None, 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599]
    ['ELMT_3401633', None, None, 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066]
    ['ELMT_3401636', None, None, 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578]
    ['ELMT_3401637', None, None, 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293]
    ['ELMT_3401638', None, None, 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646]
    ['ELMT_3401639', None, None, 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', None, None, 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['ELMT_3401644', None, None, 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705]
    ['ELMT_3401645', None, None, 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481]
```

Here's what the same join would look like as a strictly inner join::

```python
 >>> md = Matricks(test_raw_data_3, skipcols=3)
 >>> jd = a1.join(md)
 >>> print jd.labels
    ['probe_id', 'gene', 'chromo', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']
 >>> for r in jd: print r
```

Analyzing Matricks Data
-----------------------

`Matricks` instances have a number of methods to aide in the analysis
and filtering of the data they contain.  These include tools for sorting, scoring
and determining statistical correlation between profiles.

####Sorting

The output of the examples so far has been unsorted.   In nearly all cases, however,
the user will want to order the results.
The builtin `sorted` function has been incorporated into the `Matricks` class 
(as the `sorted` method) for
just this purpose.  The method assumes the list to be sorted is the instances internal data,
but otherwise takes the same (keyword) arguments as the built-in equivalent and mainly serves
as a wrapper to this function.  The noteworthy difference is that results are returned sorted
from highest to lowest, rather than from lowest to highest.   This is because in the application
for which this was initially written, most of the time we're interested in the highest 
numbers (expression levels).  The ordering can be reversed by setting ``reverse=False``
when calling this method.

Instead of passing a list as the first argument, this method passes the sample name or
its index in the label list.  Note that sorting takes place on one and only one column,
so the sample name (i.e. label) "globbing" that is permitted for column selection in `getLabel`
or `choi_score` is not permitted here; only a single column name or index may be supplied.
If no sample is specified, the instance will be sorted on the first data column (default,
column 1.)   Here's a sorted version of our example data::

```python
 >>> a1_sorted = a1.sorted()
 >>> for r in a1_sorted: print r
    ['ELMT_3401636', 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578]
    ['ELMT_3401625', 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901]
    ['ELMT_3401645', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993]
    ['ELMT_3401637', 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293]
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886]
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066]
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019]
    ['ELMT_3401632', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599]
    ['ELMT_3401644', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705]
    ['ELMT_3401619', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436]
    ['ELMT_3401623', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381]
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118]
    ['ELMT_3401614', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577]
    ['ELMT_3401612', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585]
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578]
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646]
```

Notice two things:  first, since we didn't specify a sort column, it used the first column
of numeric data for sorting.  Secondly, the sort order is from highest to lowest.  The application
for which this was originally written was typically concerned with higher values in the dataset
or analysis results.  Specifying ``reversed=False`` in the keyword arguments to the `sorted` method
will cause it to return results ordered low-to-high.

####Aggregation

Datasets often contain columns that are closely related and which can be aggregated together
somehow into a single column.      
To satisfy this need, `Matricks` provides
the ``aggregate`` method.  This will take a dictionary or function that maps sample names in
the current instance to new sample names in a new instance.  The corresponding columns will be
aggregated using the optional function provided with ``agg_fn=`` keyword argument, or the default
(arithmetic mean of non-nulls) aggregator function.

Suppose our demo data pertains to cell types and the individual replicates are
denoted by the parenthesized numbers suffixes.  We can derive the cell type
names from the sample names using a regular expression::

```python
 >>> cell_type_pat = re.compile(r'([\w_\-\+\.]+)(\(\w+\))?')
```

We can then embedd this (compiled) pattern into a function that we'll pass 
to `aggregate`::

```python
 >>> cell_type = lambda s : cell_type_pat.match(s).group(1) if cell_type_pat.match(s) is not None else s
 >>> ag1 = a1.aggregate(cell_type)
```

Since we didn't provide an alternat aggregator function, the arithmetic mean is used by default.  Notice that
the columns are in not necessarily in the same order as they were in the source instance.  The do none the less
correspond, positionally, to the labels::

```python
 >>> print ag1.getLabels()
    ['probe_id', 'Jovi', 'ABC', 'F774Lin', 'DEFC0N', 'CDDB4']

 >>> for r in ag1: print r
    ['ELMT_3401602', 6.63021747852, 4.414715071145, 6.57620493019, 6.6022379846, 6.6318799406]
    ['ELMT_3401603', 6.67341553263, 4.16456113405, 6.66361340118, 6.63635120703, 6.70026291599]
    ['ELMT_3401605', 8.70271863949, 5.50852320183, 9.39432724993, 9.7365656865, 8.88887581915]
    ['ELMT_3401607', 7.05207191931, 4.16676649019, 6.96818993978, 6.75639196465, 6.70203184527]
    ['ELMT_3401612', 6.51635952774, 4.132950304195, 6.66963293585, 7.05322172216, 6.62893626542]
    ['ELMT_3401614', 6.5067348291, 4.139474674415, 6.53686489577, 6.65934616753, 6.54032931162]
    ['ELMT_3401619', 6.78383553317, 4.17283584353, 7.26045576436, 6.67986657646, 6.57837221187]
    ['ELMT_3401623', 6.6403692878, 4.16913829652, 6.67254761381, 6.80554955104, 6.64764879202]
    ['ELMT_3401625', 9.84545402073, 6.1654989392, 10.0755468901, 10.3501936853, 9.26241934526]
    ['ELMT_3401626', 11.2541801046, 4.60661017045, 7.11119485886, 8.10062767869, 9.24637465474]
    ['ELMT_3401628', 6.68541200426, 4.120381659085, 6.53825496578, 6.63538974978, 6.66347280086]
    ['ELMT_3401632', 6.67796164216, 4.327674187545, 7.47303945599, 6.59048802252, 6.68431036403]
    ['ELMT_3401633', 7.40407740412, 4.44209809391, 8.28373011066, 7.33123060039, 6.93376501527]
    ['ELMT_3401636', 10.7344737293, 6.58344009325, 12.3247525578, 11.7592497692, 10.8845553078]
    ['ELMT_3401637', 9.69594817225, 5.41944818303, 10.7308783293, 9.93949907204, 8.84061428541]
    ['ELMT_3401638', 13.5199773001, 0.8410795, 13.4779333646, 0.0, 13.4536343874]
    ['ELMT_3401639', 6.68270691586, 4.457135884225, 6.69342317943, 6.95198458669, 6.96898611023]
    ['Elmt_3401639', 6.68270691586, 4.457135884225, 6.69342317943, 6.95198458669, 6.96898611023]
    ['ELMT_3401644', 6.63139625302, 4.173378945305, 6.72401222705, 6.65469610536, 6.59303032509]
    ['ELMT_3401645', 8.4171269045, 5.58489159156, 6.65231345481, 8.8286277291, 7.66907923624]
```

####Scoring

The `scored` method supports the creation of a score for each row in the instance
which can  then be used to sort or  ignore the rows to create a set of rows that
will constitute an result `Matricks` instance.  

Scoring is accomplished through the use of `Scorer` classes.  These classes have
a constructor and a *call* method.  The constructor can be passed parameters that
will initiate the `Scorer` instance, including the `Matricks` instance itself.
The `call` method will be invoked by the `Matricks` instance's `scored` method in
a loop in which each row is passed to the `call` method and the (scalar) result
recorded by appending it to a copy of that row to form the additional column.

There are two `Scorer` classes that are included with this package in *scoring* 
module:  `Choi`, and `GodelPositional`.

**Choi scoring** is named after Dr. Jarny Choi, who first employed it as a 
fast way to 
score expression profiles in the genome exploration tool `GuIDE`, which
he developed at the `Walter and Eliza Hall Institute`_
in 
Melbourne.   It uses structured group notation 
to specify low and high groups with each expression profile and then
culls these into vectors from which the highest of the lows
is subtracted from the lowest of the highs to obtain the score
for that profile.

.. _Walter and Eliza Hall Institute: http://www.wehi.edu.au/

If no low or high is specified, the entire sample (label) list is used.

The result is a `Matricks` instance that includes the probe ID
column(s) and the score column. 

Here is a trivial example, which specifies no low or high group:: 

```python
 >>> #cs1 = a1.choi_score()
 >>> cs1 = a1.scored(Choi(a1), label='choi score', cleanup=False, modal=False)
 >>> print cs1.getLabels()
    ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)', 'choi score']

 >>> if len(cs1) > 0: 
 ...     for r in cs1: print r
    ['ELMT_3401614', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577, -4.9771871675299995]
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578, -5.003253004259999]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118, -5.01810391599]
    ['ELMT_3401644', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705, -5.04185322705]
    ['ELMT_3401623', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381, -5.12339055104]
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978, -5.36991291931]
    ['ELMT_3401612', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585, -5.37106272216]
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019, -5.46511214229]
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943, -5.549953768450001]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943, -5.549953768450001]
    ['ELMT_3401619', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436, -5.578296764359999]
    ['ELMT_3401632', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599, -5.790880455990001]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066, -6.60157111066]
    ['ELMT_3401645', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481, -7.805465183119999]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993, -8.0544066865]
    ['ELMT_3401625', 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901, -8.966679878399999]
    ['ELMT_3401637', 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293, -9.048719329299999]
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886, -9.5720211046]
    ['ELMT_3401636', 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578, -10.6425935578]
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646, -13.5199773001]
```

A threshhold may be specified that will limit the rows returned to only those
for which the score falls at or above the threshhold.  Otherwise, all rows
are returned.

```python
 >>> #cs1 = a1.choi_score(thresh=-7.0)
 >>> cs1 = a1.scored(Choi(a1), label='choi score', thresh=-7.0, cleanup=False, modal=False)
 >>> print cs1.getLabels()
    ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)', 'choi score']

 >>> for r in cs1: 
 ...     print r
    ['ELMT_3401614', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577, -4.9771871675299995]
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578, -5.003253004259999]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118, -5.01810391599]
    ['ELMT_3401644', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705, -5.04185322705]
    ['ELMT_3401623', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381, -5.12339055104]
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978, -5.36991291931]
    ['ELMT_3401612', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585, -5.37106272216]
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019, -5.46511214229]
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943, -5.549953768450001]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943, -5.549953768450001]
    ['ELMT_3401619', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436, -5.578296764359999]
    ['ELMT_3401632', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599, -5.790880455990001]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066, -6.60157111066]
```

Now let's specify a low and high group.   We'll use the LSK samples
for our low group and the rest of the samples for the high, which we can obtain
using the `modulus` operator::

```python
 >>> lows = a1.getLabels('ABC.*', re=True)
 >>> #highs = (a1 % a1[lows]).getLabels()[1:]  # don't want the "probe_id" label
 >>> print lows
    [['ABC(12)', 'ABC(13)']]
 
 >>> #print highs
    ['CDDB4(5)', 'DEFC0N(1)', 'Jovi(1)', 'F774Lin(3)']
```

Note that when there are subgroupings of samples, an aggregator
function is applied -- recursively, if need be -- to the group
to reduce it to a scalar that will represent the group.  Thus,
the low and high lists to which ``max`` and ``min`` are applied
will always be lists of scalars.  The default aggregator
function simply takes the arithmetic mean of fht **non-null**
elements in the group.   The ``agg=`` keyword argument may
be used to pass a different aggregator function either to
this method, or to the `Matricks` constructor. 

The Choi score for this is::

```python
 >>> #cs2 = a1.scored(Choi(a1, low=lows, high=highs), label='choi score', cleanup=False, modal=False)
 >>> #for r in cs2: print r
    ['ELMT_3401636', 11.4847211865, 1.682159, 11.7592497692, 10.8845553078, 10.7344737293, 12.3247525578, 4.151033636049999]
    ['ELMT_3401637', 9.15673736606, 1.682159, 9.93949907204, 8.84061428541, 9.69594817225, 10.7308783293, 3.42116610238]
    ['ELMT_3401605', 9.33488740366, 1.682159, 9.7365656865, 8.88887581915, 8.70271863949, 9.39432724993, 3.1941954376599995]
    ['ELMT_3401625', 10.6488388784, 1.682159, 10.3501936853, 9.26241934526, 9.84545402073, 10.0755468901, 3.0969204060599997]
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978, 2.53526535508]
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886, 2.5045846884100005]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066, 2.4916669213599993]
    ['ELMT_3401603', 6.6469632681, 1.682159, 6.63635120703, 6.70026291599, 6.67341553263, 6.66361340118, 2.4717900729799993]
    ['ELMT_3401623', 6.65611759304, 1.682159, 6.80554955104, 6.64764879202, 6.6403692878, 6.67254761381, 2.4712309912799997]
    ['ELMT_3401644', 6.66459889061, 1.682159, 6.65469610536, 6.59303032509, 6.63139625302, 6.72401222705, 2.419651379785]
    ['ELMT_3401628', 6.55860431817, 1.682159, 6.63538974978, 6.66347280086, 6.68541200426, 6.53825496578, 2.4178733066950002]
    ['ELMT_3401619', 6.66351268706, 1.682159, 6.67986657646, 6.57837221187, 6.78383553317, 7.26045576436, 2.40553636834]
    ['ELMT_3401612', 6.58374160839, 1.682159, 7.05322172216, 6.62893626542, 6.51635952774, 6.66963293585, 2.3834092235449997]
    ['ELMT_3401614', 6.59679034883, 1.682159, 6.65934616753, 6.54032931162, 6.5067348291, 6.53686489577, 2.367260154685]
    ['ELMT_3401632', 6.97318937509, 1.682159, 6.59048802252, 6.68431036403, 6.67796164216, 7.47303945599, 2.2628138349749998]
    ['ELMT_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943, 2.225571031635]
    ['Elmt_3401639', 7.23211276845, 1.682159, 6.95198458669, 6.96898611023, 6.68270691586, 6.69342317943, 2.225571031635]
    ['ELMT_3401602', 7.14727114229, 1.682159, 6.6022379846, 6.6318799406, 6.63021747852, 6.57620493019, 2.161489859045]
    ['ELMT_3401645', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481, 1.0674218632499999]
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646, -0.8410795]
```

** |godel| Positional Scoring** uses |godel| numbering to characterize the pattern of high
and low values across a row as an integer.  The *positional* aspect derives from
the ability of these numbers to indicate the positions, within the row, where
the (relative) low and high values are found.   Rows with the same "fingerprint" will have highs
and lows in the same columns, though the exact values of these *extrema* may, in fact,
differ.   Two rows may each both have two "peaks", but their |godel| numbers will only
be the same if these peaks occur in the same columns for both rows.  
For each row, the mean and standard deviation are determined.  Highs and lows are
then found by seeing of the distance of a given value from the mean exceeds some multiple
of the standard deviation for the row.  Rows that have more than a specified number of extrema
are considered uninteresting or `flat-liners` and their score will be ``None``.

This makes |godel| Positional Scoring (GPS) useful for creating heatmaps.  A `Matricks` instance
sorted using GPS can then be used to create a heatmap in which similarly "shaped" profiles will
be clustered together.

Here is a |godel| positional scoring example for our test `Matricks`::

```python
 >>> gs = a1.scored(GodelPositional(a1, .8, 4), label='godel score', cleanup=False, modal=False)
 >>> print gs.getLabels()
   ['probe_id', 'ABC(12)', 'ABC(13)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)', 'godel score']

 >>> for r in gs:  print r
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646, 30060030]
    ['ELMT_3401633', 7.20203718782, 1.682159, 7.33123060039, 6.93376501527, 7.40407740412, 8.28373011066, 507]
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886, 363]
    ['ELMT_3401645', 9.48762418312, 1.682159, 8.8286277291, 7.66907923624, 8.4171269045, 6.65231345481, 12]
```

If we were to depict the pattern of highs and lows in a rough ASCII-art pattern, it would look like this::

```
 ++++++++++++##
 ++++++++++++##
 ##++##....++##
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ##++########++
 ++++++######++
```

It is readily seen how similarly-shaped rows are now clustered together.

```python
 >>> hlp = a1.scored(HiLoPositional(a1, .5, 8), cleanup=False, modal=False)
 >>> for r in hlp:  print r
    ['ELMT_3401607', 6.65137398038, 1.682159, 6.75639196465, 6.70203184527, 7.05207191931, 6.96818993978, 'Jovi(1):F774Lin(3):DEFC0N(1):CDDB4(5):ABC(12):ABC(13):Jovi(1):F774Lin(3):DEFC0N(1):CDDB4(5):ABC(12):ABC(13)']
    ['ELMT_3401638', 0.0, 1.682159, 0.0, 13.4536343874, 13.5199773001, 13.4779333646, 'Jovi(1):F774Lin(3):CDDB4(5):ABC(13):ABC(12):DEFC0N(1)']
    ['ELMT_3401626', 7.5310613409, 1.682159, 8.10062767869, 9.24637465474, 11.2541801046, 7.11119485886, 'Jovi(1):CDDB4(5):DEFC0N(1):ABC(12):F774Lin(3):ABC(13):Jovi(1):CDDB4(5):DEFC0N(1):ABC(12):F774Lin(3):ABC(13)']
    		     :

 >>> x11 = Matricks([[ 'a',  'b',    'c',  'd'], 
 ...                 ['r1',  'r1.1',   4,   5], 
 ...                 ['r2',  'r2.2',  10,  11], 
 ...                 ['r3',  'r3.1',  21,  31]],
 ... skipcols=2, cvt=float)

 >>> print x11.getLabels()
    ['a', 'b', 'c', 'd']

 >>> for r in x11: print r
    ['r1', 'r1.1', 4.0, 5.0]
    ['r2', 'r2.2', 10.0, 11.0]
    ['r3', 'r3.1', 21.0, 31.0]

 >>> a1t = x11.transpose()
 >>> for r in a1t: print r
    ['c', 4.0, 10.0, 21.0]
    ['d', 5.0, 11.0, 31.0]

 >>> a1t2 = a1t.transpose()
 >>> print a1t2.getLabels()
    ['a', 'b', 'c', 'd']

 >>> for r in a1t2: print r
    ['r1', 'r1.1', 4.0, 5.0]
    ['r2', 'r2.2', 10.0, 11.0]
    ['r3', 'r3.1', 21.0, 31.0]

```

####Pearson Product Moment Correlation

One of the calculations often performed with expression profiles is
the `Pearson Product Moment Coefficient`_. We can supply any number of profile identifiers 
(probe IDs; not to be confused with sample names or labels)
used to measure the degree to which a given profile correlates to other profiles
within the dataset.

.. _Pearson Product Moment Coefficient: http://en.wikipedia.org/wiki/Pearson_product-moment_correlation_coefficient/

and the result will be comprised of the same profile IDs as the parent instance, but the columns will
be the PPMCs for each of the specified profiles.  For example::

```python
 >>> ppmc1 = a1.pearson('ELMT_3401603', 'ELMT_3401607')
 >>> print ppmc1.getLabels()
    ['probe_id', 'ELMT_3401603', 'ELMT_3401607']

 >>> for r in ppmc1: print r
    ['ELMT_3401602', 0.994093822093825, 0.9873711032775128]
    ['ELMT_3401603', 1.0, 0.9973005780642519]
    ['ELMT_3401605', 0.9916915617370952, 0.986782079274304]
    ['ELMT_3401607', 0.9973005780642521, 1.0]
    ['ELMT_3401612', 0.99514309197709, 0.9911254283033979]
    ['ELMT_3401614', 0.9993463832853486, 0.995628190154687]
    ['ELMT_3401619', 0.9931235819907885, 0.9960653003415773]
    ['ELMT_3401623', 0.9992656673629212, 0.9963006286019747]
    ['ELMT_3401625', 0.9891293485234534, 0.9859142020756926]
    ['ELMT_3401626', 0.888584061331424, 0.8993865912698161]
    ['ELMT_3401628', 0.9996660578403657, 0.9971818078548389]
    ['ELMT_3401632', 0.9883887894688987, 0.9888071562877305]
    ['ELMT_3401633', 0.9811302954812443, 0.9878631745738885]
    ['ELMT_3401636', 0.9885386079431606, 0.9868884611713656]
    ['ELMT_3401637', 0.9794743559750448, 0.9871172256429301]
    ['ELMT_3401639', 0.9951083808697037, 0.9861344481367755]
    ['Elmt_3401639', 0.9951083808697037, 0.9861344481367755]
    ['ELMT_3401644', 0.9995879246226951, 0.9975119140885805]
    ['ELMT_3401645', 0.9368679872321245, 0.92376912222379]
```

Let's say we want to select only those profiles that have correlation coefficients 
that are greater than 0.99::

```python
 >>> for r in ppmc1[0.99:]: print r
    ['ELMT_3401602', 0.994093822093825, None]
    ['ELMT_3401603', 1.0, 0.9973005780642519]
    ['ELMT_3401605', 0.9916915617370952, None]
    ['ELMT_3401607', 0.9973005780642521, 1.0]
    ['ELMT_3401612', 0.99514309197709, 0.9911254283033979]
    ['ELMT_3401614', 0.9993463832853486, 0.995628190154687]
    ['ELMT_3401619', 0.9931235819907885, 0.9960653003415773]
    ['ELMT_3401623', 0.9992656673629212, 0.9963006286019747]
    ['ELMT_3401625', None, None]
    ['ELMT_3401626', None, None]
    ['ELMT_3401628', 0.9996660578403657, 0.9971818078548389]
    ['ELMT_3401632', None, None]
    ['ELMT_3401633', None, None]
    ['ELMT_3401636', None, None]
    ['ELMT_3401637', None, None]
    ['ELMT_3401639', 0.9951083808697037, None]
    ['Elmt_3401639', 0.9951083808697037, None]
    ['ELMT_3401644', 0.9995879246226951, 0.9975119140885805]
    ['ELMT_3401645', None, None]
```

This isn't really what we want.
Recall that when range is specified -- with or without sample names -- the range is
applied to *ALL* the samples specified (or to all the samples in the instance of none
are explicitly provided), and *ALL* of the samples in the instance are returned, not
just the one(s) specified, if any.  We could use the ``purge`` method, but this would
only eliminate the rows that have nulls all the way across.  The second row, for instance,
would still be included, even though one of the values is ``None``.

Without first narrowing the set of samples
to just the one(s) we're interested in, *any* profile that has a value that falls in
the specified range will be included.   We need to make use of slice semantics from above, 
to carve out the set of correlation 
coeeficients for an individual profile -- as well as a specific range of them -- 
from this result.  Suppose we wanted just
the coefficients for `ELMT_3401607` that were higher than 0.99::

```python
 >>> e607 = Matricks(ppmc1)['probe_id', 'ELMT_3401607']
 >>> print e607.getLabels()
    ['probe_id', 'ELMT_3401607']

 >>> for r in e607[:0.99]: print r
    ['ELMT_3401602', None]
    ['ELMT_3401603', 0.9973005780642519]
    ['ELMT_3401605', None]
    ['ELMT_3401607', 1.0]
    ['ELMT_3401612', 0.9911254283033979]
    ['ELMT_3401614', 0.995628190154687]
    ['ELMT_3401619', 0.9960653003415773]
    ['ELMT_3401623', 0.9963006286019747]
    ['ELMT_3401625', None]
    ['ELMT_3401626', None]
    ['ELMT_3401628', 0.9971818078548389]
    ['ELMT_3401632', None]
    ['ELMT_3401633', None]
    ['ELMT_3401636', None]
    ['ELMT_3401637', None]
    ['ELMT_3401639', None]
    ['Elmt_3401639', None]
    ['ELMT_3401644', 0.9975119140885805]
    ['ELMT_3401645', None]
```

Now can eliminate all the null rows with the ``purge=True`` setting
in the constructor, or by invoking the `purge` method::

```python
 >>> for r in e607[0.99:].purge(): print r
    ['ELMT_3401603', 0.9973005780642519]
    ['ELMT_3401607', 1.0]
    ['ELMT_3401612', 0.9911254283033979]
    ['ELMT_3401614', 0.995628190154687]
    ['ELMT_3401619', 0.9960653003415773]
    ['ELMT_3401623', 0.9963006286019747]
    ['ELMT_3401628', 0.9971818078548389]
    ['ELMT_3401644', 0.9975119140885805]
```

####Distance

The `distance` method will return the distance between two rows in a `Matricks` instance
as calculated by the selected function.  The default distance function is the 
`Pearson Product Moment Correlation`, but there are several others.  This method
more or less mirrors the distance functions described in *The C Clustering Library* 
(Hoon, Imoto, Miyano -- Univ. of Tokyo).  (See `distance` method documentation, below.)

Here are several examples::

```python
 >>> print round(a1.distance(1, 2), 4)        # Pearson correlation (default -- 'c')
    0.0083

 >>> print round(a1.distance(1, 2, 'a'), 4)   # Absolute Pearson
    0.0083

 >>> print round(a1.distance(1, 2, 'u'), 4)   # Uncentered Pearson
    0.0014

 >>> print round(a1.distance(1, 2, 'x'), 4)   # Absolute Uncentered Pearson
    0.0014

 >>> print round(a1.distance(1, 2, 's'), 4)   # Spearman's Rank Correlation
    1.1429

 >>> print round(a1.distance(1, 2, 'k'), 4)   # Kendall's Tau (Spearman variation)
    0.1333

 >>> print round(a1.distance(1, 2, 'e'), 4)   # Euclidean
    5.5335

 >>> print round(a1.distance(1, 2, 'b'), 4)   # City Block (aka Manhattan) 
    2.1228
```

####Detection Summary

If we want to know which samples contain values that fall above some threshhold, we can use the
`detection_summary` method.   The algorithm this uses is as follows:

1. Given a list of profiles, construct a list that is the maximum value for each sample
   across this set.

2. Construct a list of sample names that includes the names of samples for which the
   corresponding maximum value (from step 1) exceeds a given threshhold.   If no threshhold is
   specified, use the dataset-wide mean value.


Example::

```python
 >>> print a1.detection_summary('ELMT_3401602','ELMT_3401603','ELMT_3401605','ELMT_3401607')
     ['ABC(12)', 'DEFC0N(1)', 'CDDB4(5)', 'Jovi(1)', 'F774Lin(3)']
```

Using *Matricks* as a Base Class
--------------------------------

It's easy to use *Matricks* as a base class, but it's important to remember that any superclass 
must provide its own constructor that calls *Matricks* constructor.   A very minimal example::

```python

 >>> class MyMat (Matricks):
 ...     def __init__(self, *args, **kwargs):
 ...         super(MyMat, self).__init__(*args, **kwargs)
 
 >>> m_inst = MyMat(test_raw_data, cvt=lambda x: float(x) if len(x) > 0 else None)
 >>> print m_inst
    [['probe_id' 'ABC(12)' 'ABC(13)' ... 'CDDB4(5)' 'Jovi(1)' 'F774Lin(3)'],
      ['ELMT_3401602' 7.14727114229 1.682159 ... 6.6318799406 6.63021747852 6.57620493019],
      ['ELMT_3401603' 6.6469632681 1.682159 ... 6.70026291599 6.67341553263 6.66361340118],
      ['ELMT_3401605' 9.33488740366 1.682159 ... 8.88887581915 8.70271863949 9.39432724993],
    ...  
      ['Elmt_3401639' 7.23211276845 1.682159 ... 6.96898611023 6.68270691586 6.69342317943],
      ['ELMT_3401644' 6.66459889061 1.682159 ... 6.59303032509 6.63139625302 6.72401222705],
      ['ELMT_3401645' 9.48762418312 1.682159 ... 7.66907923624 8.4171269045 6.65231345481]]

```

You can now pickle this instance and, when unpickled, it will be an instance of ``MyMat``.
If you don't include the ``__init__`` method, or the call to ``super`` therein, the results
will most likely be not what you wanted or expected.
