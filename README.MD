Ontario Bridges

The Government of Ontario collects [a huge amount of data](https://data.ontario.ca/) on provincial programs and infrastructure, and much of it is provided as open data sets for public use. This project works with a particular dataset that contains information about provincially owned and maintained bridges in Ontario. All bridges in Ontario are reviewed every 2 years, and their information is collected to help determine when bridges need inspection and maintenance. The data contains information about all bridges in the Ontario highway network, such as the length of the bridges, their condition over various years, and historical information.

The datasets in starter files: bridge_data_small.csv and bridge_data_large.csv. These are Comma Separated Value (CSV) files, which contain the data in a table-format similar to a spreadsheet. Each row of the table contains information about a bridge. And each column of the table represents a “feature” of that bridge. For example, the latitude and longitude columns tell us the precise location of the bridge.

Here is a screenshot of the data opened in Microsoft Excel (your computer may have a different spreadsheet program installed):





We can see that the first bridge is described on row 3. From row 3, column B, we can see that the bridge is named: Highway 24 Underpass at Highway 403. Subsequent columns include even more information about the bridge.

Inspecting Bridges
Ontario sends inspectors to check the condition of a bridge. The dataset contains a column (LAST INSPECTION DATE) showing the last date a bridge was inspected. When a bridge is inspected, it receives a score based on its condition. This score is called the Bridge Condition Index (BCI). The BCI is a number between 0 and 100, inclusive. You can see the most recent score in the dataset (CURRENT BCI), as well as past scores (the columns with years ranging from 2013 to 2000).

Fixing Bridges

If a bridge is in poor condition, it can be fixed (i.e., “rehabilitated”). These can be major or minor fixes. The dataset includes the year the last major (LAST MAJOR REHAB) or minor (LAST MINOR REHAB) rehabilitation was performed on the bridge.

Bridge Spans

A bridge is made up of one or more spans (# OF SPANS). A span “is the distance between two intermediate supports for a structure, e.g. a beam or a bridge. A span can be closed by a solid beam or by a rope” (Source: Wikipedia). Each span has a length associated with it (SPAN DETAILS). For example, if a bridge has two spans, the SPAN DETAILS data follows the following format:

Total=60.6  (1)=30.3;(2)=30.3;
More generally, the format is:

Total=[total length of all spans] (1)=[the length of the first span];(2)=[the length of the second span]; and so on for each span of the bridge;
Some things to notice about this format:

There is no semicolon after the total length
There is a semicolon after every span length
Each span length has a prefix of the form (x)= where x is a number starting from 1 and increasing by 1 for every span.


Ontario Bridges in Python
We will represent the dataset as a list of lists in Python (i.e., List[list]). The outer list has the same length as the number of bridges in the dataset. Each inner list (i.e., list) corresponds to one row (i.e., bridge) of the dataset. For example, here is what the first bridge of our dataset will look like in our Python program:

>>> MISSING_BCI = -1.0
>>> first_bridge = [
...     1, 'Highway 24 Underpass at Highway 403',
...     '403', 43.167233, -80.275567, '1965', '2014', '2009', 4,
...     [12.0, 19.0, 21.0, 12.0], 65.0, '04/13/2012',
...     [['2013', '2012', '2011', '2010', '2009', '2008', '2007',
...       '2006', '2005', '2004', '2003', '2002', '2001', '2000'],
...      [MISSING_BCI, 72.3, MISSING_BCI, 69.5, MISSING_BCI, 70.0, MISSING_BCI,
...       70.3, MISSING_BCI, 70.5, MISSING_BCI, 70.7, 72.9, MISSING_BCI]]
... ]
The variable first_bridge has the general type list. Notice how the elements inside the list are not all the same type. The list includes:

integers (e.g., 1, 4)
strings (e.g., '403', '04/13/2012')
floats (e.g., 65.0)
lists (e.g., [12.0, 19.0, 21.0, 12.0])
You may also notice that first_bridge is different from the first bridge in the dataset file itself. This is because the data has been cleaned to suit our needs. For example:

we have replaced the ID with an integer value.
we have replaced the spans with a list of floats, omitting the total ([12.0, 19.0, 21.0, 12.0])
we have replaced the BCI scores with two lists. The first is a list of strings containing the dates. The second is a parallel list (i.e., has the same length as the list of strings) of floats containing the scores, where empty scores are assigned the value MISSING_BCI
The data is not magically converted into this “clean” format - you will be implementing functions that transform the text data found in the files into a format that is more useful to our program.

Indexing with Constants

The bridge_functions.py file includes many constants to use when indexing the nested data. Much like Assignment 1, you should be using these constants in the bodies of your functions. For example, what should we write if we wanted to access the year a bridge was built?

Consider the following code that does not use the constants:

>>> # Assume that first_bridge is a list containing data
>>> first_bridge[5]
'1965'
How did I know to use index 5? Am I expected to memorize all the indexes? The answer is no; you should not write code like above. Instead, use the constants to provide context into which data feature you are accessing:

>>> # Assume that first_bridge is a list containing data
>>> first_bridge[COLUMN_YEAR_BUILT]
'1965'
The following table shows how the dataset file and Python data are related through constants that begin with the prefix COLUMN_. The table also includes the data type that you should expect to find if you were to index a bridge list (like first_bridge) using that constant.

Column Name	Constant to use as Index	Data Type
ID	COLUMN_ID	int
STRUCTURE	COLUMN_NAME	str
HWY NAME	COLUMN_HIGHWAY	str
LATITUDE	COLUMN_LAT	float
LONGITUDE	COLUMN_LON	float
YEAR BUILT	COLUMN_YEAR_BUILT	str
LAST MAJOR REHAB	COLUMN_LAST_MAJOR_REHAB	str
LAST MINOR REHAB	COLUMN_LAST_MINOR_REHAB	str
# OF SPANS	COLUMN_NUM_SPANS	int
SPAN DETAILS	COLUMN_SPAN_DETAILS	List[float]
DECK LENGTH	COLUMN_DECK_LENGTH	float
LAST INSPECTION DATE	COLUMN_LAST_INSPECTED	str
CURRENT BCI	N/A	N/A
Remaining Columns	COLUMN_BCI	List[list]
Note that the COLUMN_ID in our inner list is an integer, which is very different from the ID column in the dataset.

Storing BCI Scores

Our inner list does not contain the CURRENT BCI column from the dataset (instead, you will implement a function to find the most recent BCI score). Moreover, the remaining columns in the dataset that contain BCI scores are stored in another list (at index COLUMN_BCI) with type List[list]. This list contains exactly two lists:

The first list is at INDEX_BCI_YEARS with type List[str] and includes the years in decreasing order.
The second list is at INDEX_BCI_SCORES with type List[float] and includes the BCI scores. Empty scores in the dataset have a value of MISSING_BCI.
These two lists are parallel lists and should have the same length. Consider the following example:

>>> # Assume that first_bridge is a list containing data
>>> # Assume that MISSING_BCI refers to the value -1.0
>>> first_bridge[COLUMN_BCI][INDEX_BCI_YEARS]
['2013', '2012', '2011', '2010', '2009', '2008', '2007', '2006', '2005', '2004', '2003', '2002', '2001', '2000']
>>> first_bridge[COLUMN_BCI][INDEX_BCI_SCORES]
[-1.0, 72.3, -1.0, 69.5, -1.0, 70.0, -1.0, 70.3, -1.0, 70.5, -1.0, 70.7, 72.9, -1.0]
>>> len(first_bridge[COLUMN_BCI][INDEX_BCI_YEARS])
14
>>> len(first_bridge[COLUMN_BCI][INDEX_BCI_SCORES])
14
From the example above, we can see that first_bridge has no BCI score in the year 2013 (see index 0 of both lists). But it does have a BCI score of 72.3 in the year 2012 (see index 1 of both lists). Therefore, first_bridge’s most recent BCI score is 72.3.

Locations and Calculating Distance

The bridges have their locations represented as a latitude and longitude, which we typically refer to as (lat, lon) for short. If you are curious, you can always search for a specific location online (e.g., with Google Maps):







It is very convenient to be able to calculate the straight-line distance between two locations. But this is actually a little tricky due to the curvature of the earth. We are providing you with the full implementation of a function, calculate_distance, that will accurately return the distance between two (lat, lon) points. You do not need to know how this function works - you only need to know how to use it.
