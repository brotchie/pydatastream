# PyDatastream

PyDatastream is a Python interface to the [Thomson Dataworks Enterprise](http://dataworks.thomson.com/Dataworks/Enterprise/1.0/) (DWE) SOAP API (non free), with some convenience functions for retrieving Datastream data specifically. This package requires valid credentials for this API.

## Notes

* This package is mainly meant to access Datastream. However basic functionality (```request``` method) should work for [other Dataworks Enterprise sources](http://dtg.tfn.com/data/).
* The package is using [Pandas](http://pandas.pydata.org/) library ([GitHub repo](https://github.com/pydata/pandas)), which I found to be the best Python library for time series manipulations. Together with [IPython notebook](http://ipython.org/notebook.html) it is the best open source tool for the data analysis. For quick start with pandas have a look on [tutorial notebook](http://nbviewer.ipython.org/urls/gist.github.com/fonnesbeck/5850375/raw/c18cfcd9580d382cb6d14e4708aab33a0916ff3e/1.+Introduction+to+Pandas.ipynb) and [10-minutes introduction](http://pandas.pydata.org/pandas-docs/stable/10min.html).
* Alternatives for other scientific computing languages:
  - MATLAB: [MATLAB datafeed toolbox](http://www.mathworks.fr/help/toolbox/datafeed/datastream.html)
  - R: [RDatastream](https://github.com/fcocquemas/rdatastream) (in fact PyDatastream was inspired by RDatastream).
* I am always open for suggestions, critique and bug reports.

## Installation

First, install prerequisites: `pandas` and `suds`. Both of packages can be installed with the [pip installer](http://www.pip-installer.org/en/latest/):

    pip install pandas
    pip install suds

However please refer to the [pandas documentation](http://pandas.pydata.org/pandas-docs/stable/install.html) for the dependencies. 

....

## Basic use

All methods to work with the DWE is organized as a class, so first you need to create an object with your valid credentials:

    from pydatastream import Datastream
    DWE = Datastream(username="DS:XXXX000", password="XXX000")

If authentication was successfull, then you can check system information (including version of DWE):

    DWE.system_info()

and list of data sources available for your subscription:

	DWE.sources()

Basic functionality of the module allows to fetch Open-High-Low-Close (OHLC) prices and Volumes for given ticker.

### Daily data

The following command requests daily closing price data for the Apple asset (DWE mnemonic `"@AAPL"`) on May 3, 2000:

	data = DWE.get_price('@AAPL', date='2000-05-03')
	print data
	
or request daily closing price data for Apple in 2008:

	data = DWE.get_price('@AAPL', date_from='2008', date_to='2009')
	print data.head()

The data is retrieved as pandas.DataFrame object, which can be plotted:

	data.plot()
	
aggregated into monthly data:

	print data.resample('M', how='last')
	
or [manipulated in a variety of ways](http://nbviewer.ipython.org/urls/raw.github.com/changhiskhan/talks/master/pydata2012/pandas_timeseries.ipynb). Due to extreme simplicity of resampling  the data in Pandas library (for example, tainkg into account business calendar), I would recommend to request daily data (unless the requests are huge or daily scale is not applicable) and perform all transformations locally. Also note that thanks to Pandas library format of the date string is extremely flexible.


For fetching Open-High-Low-Close (OHLC) data there exist two methods: `get_OHLC` to fetch only price data and `get_OHLCV` to fetch boyth price and volume data. This separation is required as volume data is not available for financial indices.

Request daily OHLC and Volume data for Apple in 2008:

	data = DWE.get_OHLCV('@AAPL', date_from='2008', date_to='2009')
	
Request daily OHLC data for S&P 500 Index from May 6, 2010 until present date:

	data = DWE.get_OHLC('S&PCOMP', date_from='May 6, 2010')

### Requesting specific fields for data

If the Thomson Reuters mnemonic for specific fields are known, then more general function can be used. The following request

	data = DWE.fetch('@AAPL', ['P','MV','VO',], date_from='2000-01-01')

fetchs the closing price, daily volume and market valuation for Apple Inc.

### Constituents list for indices

PyDatastream also has an interface for retrieving list of constituents of indices:

	(res, status) = DWE.get_constituents('S&PCOMP')
	print res.ix[0]

As an option, the list for a specific date can be requested as well:

	(res, status) = DWE.get_constituents('S&PCOMP', '1-sept-2013')

## Advanced use

The module has a general-purpose function `request` that can be used for fetching data with custom requests.

### Using custom requests

The Datastream request syntax is somewhat arcane but can be more powerful in certain cases. A [decent guide can be found here](http://dtg.tfn.com/data/DataStream.html). You can use this syntax directly with this package when your needs are more sophisticated.

For instance, let's say I want the data from the previous example combined in a single dataframe.

    request1 <- "U:IBM,@MSFT~=P,MV~2007-06-04~:2009-06-04~M"
    dat <- ds(user, requests = request1)
    dat[["Data",1]]
    
We can run several such requests in a single API call.

    request2 <- "U:MMM~=P,PO~2007-09-01~:2007-09-12~D"
    request3 <- "906187~2008-01-01~:2008-10-02~M"
    request4 <- "PCH#(U:BAC(MV))~2008-01-01~:2008-10-02~M"
    requests <- c(request1, request2, request3, request4)
    dat <- ds(user, requests = requests)
    dat["Data",]


### Debugging 

For the debugging puroposes, Datastream class has `show_request` property, which, if set to `True` makes standard methods to output the text string with request:

	DWE.show_request = True
	data = DWE.fetch('@AAPL', ['P','MV','VO',], date_from='2000-01-01')

### Other useful tips with the Datastream syntax

#### Get some reference information on a security with `"~XREF"`, including ISIN, industry, etc.

    dat <- ds(user, requests = "U:IBM~XREF") 
    dat[["Data",1]]
    
#### Get some static items like NAME, ISIN with `"~REP"`

    dat <- ds(user, requests = "U:IBM~=NAME,ISIN~REP") 
    dat[["Data",1]]

#### Convert the currency e.g. to Euro with `"~~EUR"`

    dat <- ds(user, requests = "U:IBM(P)~~EUR~2007-09-01~:2009-09-01~D") 
    dat[["Data",1]]
    
#### Use Datastream expressions, e.g. for a moving average on 20 days

    dat <- ds(user, requests = "MAV#(U:IBM,20D)~2007-09-01~:2009-09-01~D") 
    dat[["Data",1]]

## Resources

It is recommended that you read the [Thomson Dataworks Enterprise User Guide](http://dataworks.thomson.com/Dataworks/Enterprise/1.0/documentation/user%20guide.pdf), especially section 4.1.2 on client design. It gives reasonable guidelines for not overloading the servers with too intensive requests.

For building custom Datastream requests, useful guidelines are given on this somewhat old [Thomson Financial Network](http://dtg.tfn.com/data/DataStream.html) webpage.

If you have access codes for the Datastream Extranet, you can use the [Datastream Navigator](http://product.datastream.com/navigator/) to look up codes and data types.

## Licence

PyDatastream is released under the MIT licence.