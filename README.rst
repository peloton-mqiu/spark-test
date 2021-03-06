spark-test
==========

.. image:: https://travis-ci.com/tomasfarias/spark-test.svg?branch=master
    :target: https://travis-ci.com/tomasfarias/spark-test

A collection of assertion functions to test Spark Collections like DataFrames

Motivation
----------

As you develop Spark applications, you can eventually end up writing methods that apply transformations over Spark DataFrames. In order to test the results, you can create ``pandas`` DataFrames and use the test functions provided by ``pandas`` as ``pyspark`` does not provide any functions to assist with testing.

``spark-test`` provides testing functions similar to ``pandas`` but geared towards Spark Collections.

Let's say you have a function to apply some transformations on a Spark DataFrame (the full code for this example can be found in tests/test_example.py:

::

  def transform(df):
      """
      Fill nulls with 0, sum 10 to Age column and only return distinct rows
      """

      df = df.na.fill(0)
      df = df.withColumn('Age', df['Age'] + 10)
      df = df.distinct()

      return df

We can then write a test case with as many test inputs as we need and test the results with ``assert_dataframe_equal``:

::

  from spark_test.testing import assert_dataframe_equal


  def test_transform(spark, transform):

      input_df = spark.createDataFrame(
          [['Tom', 25], ['Tom', 25], ['Charlie', 24], ['Dan', None]],
          schema=['Name', 'Age']
      )

      expected = spark.createDataFrame(
          [['Tom', 35], ['Charlie', 34], ['Dan', 0]],
          schema=['Name', 'Age']
      )
      result = transform(input_df)

      assert_frame_equal(expected, result)

Of course, tests are more interesting when they fail so let's introduce a bug in our ``transform`` function:

::

  def bugged_transform(df):
      """
      Fill nulls with 0, sum 10 to Age column and only return distinct rows
      """

      df = df.na.fill(1)  # Whoops! Should be 0!
      df = df.withColumn('Age', df['Age'] + 10)
      df = df.distinct()

      return df

Passing both functions to our test using ``pytest.mark.parametize`` yields the following output with a nice message on what failed:

::

  $ pytest tests/example.py
  ============================= test session starts =============================
  platform linux -- Python 3.7.3, pytest-5.0.0, py-1.8.0, pluggy-0.12.0
  rootdir: /home/tfarias/repos/spark-test
  collected 2 items

  tests/example.py .F                                                [100%]

  ================================== FAILURES ===================================
  _______________________ test_transform[bugged_transform] ________________________

              assert left_d[key] == right_d[key], msg.format(
  >               field=key, l_value=left_d[key], r_value=right_d[key]
              )
  E           AssertionError: Values for Age do not match:
  E           Left=10
  E           Right=11


License
-------

Distributed under the MIT License.
