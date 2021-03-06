# Compliance Checker


The Compliance Checker is a python based tool for data providers to check
for completeness and community standard compliance of infrastructure - local or remote
The python module can be used as a command-line tool or as a
library that can be integrated into other software.



## Usage
Select the test you want to run from the dropdown menu. Then, either upload your dataset or provide a url to a
remote dataset (OPeNDAP) and click 'Submit'.

The output of the Compliance Checker will give you a comprehensive list of issues and the actions needed to correct them.
You may download the Compliance Checker report as a text file by clicking the 'Download Report' button

![Compliance-Checker-Web](https://github.com/Jay-Wani/compliance-checker/blob/master/Compliance.PNG)

## API

In addition to a web-based front-end for the IOOS Compliance Checker project, an API is provided for
users interested in batch processing files hosted via OPeNDAP. Details on how to use the API are
available on the Compliance Checker Web [wiki page](https://github.com/ioos/compliance-checker-web/wiki/API).

Here are a couple examples:

**HTML Output**

https://data.ioos.us/compliance/api/run?report_format=html&test=acdd&url=http://sos.maracoos.org/stable/dodsC/hrecos/stationHRMARPH-agg.ncml

**JSON Output**

https://data.ioos.us/compliance/api/run?report_format=json&test=acdd&url=http://sos.maracoos.org/stable/dodsC/hrecos/stationHRMARPH-agg.ncml


# The Compliance Checker Command Line Tool


## Concepts & Terminology

Each compliance standard is executed by a Check Suite,
which functions similar to a Python standard Unit Test.
A Check Suite runs checks against a dataset based on a metadata standard,
returning a list of Results which are then aggregated into a summary.

Each Result has a (# passed / # total) score, a weight (HIGH/MEDIUM/LOW),
a computer-readable name, an optional list of human-readable messages,
and optionally a list of child Results.

A single score is then calculated by aggregating on the names,
then multiplying the score by the weight and summing them together.

The computer-readable name field controls how Results are aggregated together - in order to prevent the overall score for a Check Suite varying on the number of variables,
it is possible to *group* Results together via the name property.
Grouped results will only add up to a single top-level entry.

See the [Development](//github.com/ioos/compliance-checker/wiki/Development) wiki page for more details on implementation.

## Installation

Check out the [Installation wiki](https://github.com/ioos/compliance-checker/wiki/Installation) for instructions on how to install.

## Command Line Usage

The compliance-checker can work against local files (`.nc` files, `.cdl`
metadata files, .xml files of SOS GetCapabilities/DescribeSensor requests)
or against remote URLs (OPeNDAP data URLs, SOS GetCapabilities/DescribeSensor URLs).

If you are aiming to check a netCDF-dump, also known as a CDL file, the file
must be named to end with a `.cdl` for the check-suite to be able to correctly
parse it's contents.

> **WARNING** The CF/ACDD checks **will access data**, so if using a remote OPeNDAP URL, please be sure the size is reasonable!

```
usage: cchecker.py [-h] [--test TEST] [--criteria [{lenient,normal,strict}]]
                   [--verbose] [--skip-checks SKIP_CHECKS]
                   [-f {text,html,json,json_new}] [-o OUTPUT] [-V] [-l]
                   [-d DOWNLOAD_STANDARD_NAMES]
                   [dataset_location [dataset_location ...]]

positional arguments:
  dataset_location      Defines the location of the dataset to be checked.

optional arguments:
  -h, --help            show this help message and exit
  --test TEST, -t TEST, --test= TEST, -t= TEST
                        Select the Checks you want to perform. Defaults to
                        'acdd' if unspecified. Versions of standards can be
                        specified via `-t <test_standard>:<version>`. If
                        `<version>` is omitted, or is "latest", the latest
                        version of the test standard is used.
  --criteria [{lenient,normal,strict}], -c [{lenient,normal,strict}]
                        Define the criteria for the checks. Either Strict,
                        Normal, or Lenient. Defaults to Normal.
  --verbose, -v         Increase output. May be specified up to three times.
  --skip-checks SKIP_CHECKS, -s SKIP_CHECKS
                        Specifies tests to skip
  -f {text,html,json,json_new}, --format {text,html,json,json_new}
                        Output format. The difference between the 'json' and
                        the 'json_new' formats is that the 'json' format has
                        the check as the top level key, whereas the 'json_new'
                        format has the dataset name(s) as the main key in the
                        output follow by any checks as subkeys. Also, 'json'
                        format can be only be run against one input file,
                        whereas 'json_new' can be run against multiple files.
  -o OUTPUT, --output OUTPUT
                        Output filename(s). If '-' is supplied, output to
                        stdout. Can either be one or many files. If one file
                        is supplied, but the checker is run against many
                        files, all the output from the checks goes to that
                        file (does not presently work with 'json' format). If
                        more than one output file is supplied, the number of
                        input datasets supplied must match the number of
                        output files.
  -V, --version         Display the IOOS Compliance Checker version
                        information.
  -l, --list-tests      List the available tests
  -d DOWNLOAD_STANDARD_NAMES, --download-standard-names DOWNLOAD_STANDARD_NAMES
                        Specify a version of the cf standard name table to
                        download as packaged version
```

## Examples

### Check a local file against CF 1.6
```
$ compliance-checker --test=cf:1.6 compliance_checker/tests/data/examples/hycom_global.nc


--------------------------------------------------------------------------------
                         IOOS Compliance Checker Report
                                  cf:1.6 check
--------------------------------------------------------------------------------
                               Corrective Actions
hycom_global.nc has 9 potential issues


                                     Errors
--------------------------------------------------------------------------------
Name                                      Reasoning
§3.2 Either long_name or standard_name    Attribute long_name or/and standard_name
is highly recommended for variable time:  is highly recommended for variable time
§4.3.1 depth is a valid vertical          vertical coordinates not defining
coordinate:                               pressure must include a positive
                                          attribute that is either 'up' or 'down'


                                    Warnings
--------------------------------------------------------------------------------
Name                                   Reasoning
§2.6.1 Global Attribute Conventions    Conventions global attribute does not
includes CF-1.6:                       contain "CF-1.6". The CF Checker only
                                       supports CF-1.6 at this time.
§2.6.2 Recommended Attributes:         institution should be defined source
                                       should be defined references should be
                                       defined
§2.6.2 Recommended Global Attributes:  global attribute history should exist
                                       and be a non-empty string
§8.1 Packed Data defined by water_u    Attributes add_offset and scale_factor
contains valid packing:                are not of type float or double.
§8.1 Packed Data defined by water_v    Attributes add_offset and scale_factor
contains valid packing:                are not of type float or double.
```

### Check a remote file against ACDD 1.3

The remote dataset url is taken from the Data URL section of an OPeNDAP endpoint.

```shell
$ compliance-checker --test=acdd:1.3 "http://sos.maracoos.org/stable/dodsC/hrecos/stationHRMARPH-agg.ncml"
```

### Write results to text file

```shell
$ compliance-checker --test=acdd:1.3 --format=text --output=/tmp/report.txt compliance_checker/tests/data/examples/hycom_global.nc
```

### Write results to JSON file

```shell
$ compliance-checker --test=acdd:1.3 --format=json --output=/tmp/report.json compliance_checker/tests/data/examples/hycom_global.nc
```

### Write results to HTML file

```shell
$ compliance-checker --test=acdd:1.3 --format=html --output=/tmp/report.html compliance_checker/tests/data/examples/hycom_global.nc
```

### Output text from multiple input files to one output file

```
$ compliance-checker --test=cf:1.6 --format text --output=/tmp/combined_output.txt compliance_checker/tests/data/examples/hycom_global.nc compliance_checker/tests/data/examples/ww3.nc
```

### Output html and text files from multiple input files (part 1)
In this case you'll get 2 files ```/tmp/combined_output.txt``` and ```/tmp/combined_output.html``` that contain cf check results for both input files because you only specified 1 output filename.
```
$ compliance-checker --test=cf:1.6 --format text --format html --output=/tmp/combined_output.txt compliance_checker/tests/data/examples/hycom_global.nc compliance_checker/tests/data/examples/ww3.nc
```

### Output html and text files from multiple input files (part 2)
In this case you'll get 4 files ```/tmp/hycom.txt```, ```/tmp/hycom.html```, ```/tmp/ww3.txt```, and ```/tmp/ww3.html``` that contain cf check results because you specified as many output filenames as input filenames.
```
$ compliance-checker --test=cf:1.6 --format text --format html --output=/tmp/hycom.txt --output=/tmp/ww3.txt compliance_checker/tests/data/examples/hycom_global.nc compliance_checker/tests/data/examples/ww3.nc
```

### Download a particular CF standard names table for use in the test

**Note**
During the CF test, if a file has a particular version of the cf standard name table specified in the global attributes
(i.e. ```:standard_name_vocabulary = "CF Standard Name Table v30" ;```) that doesn't match the packaged version, it will
try to download the specified version. If it fails, it will fall back to packaged version.

```
$ compliance-checker -d 35

Downloading cf-standard-names table version 35 from: http://cfconventions.org/Data/cf-standard-names/35/src/cf-standard-name-table.xml
```



## Python Usage

If you are interested in incorporating the IOOS Compliance Checker into your own python projects, check out the following python code example:
```python
from compliance_checker.runner import ComplianceChecker, CheckSuite

# Load all available checker classes
check_suite = CheckSuite()
check_suite.load_all_available_checkers()

# Run cf and adcc checks
path = '/path/or/url/to/your/dataset'
checker_names = ['cf', 'acdd']
verbose = 0
criteria = 'normal'
output_filename = '/output/report.json'
output_format = 'json'
"""
Inputs to ComplianceChecker.run_checker

path            Dataset location (url or file)
checker_names   List of string names to run, should match keys of checkers dict (empty list means run all)
verbose         Verbosity of the output (0, 1, 2)
criteria        Determines failure (lenient, normal, strict)
output_filename Path to the file for output
output_format   Format of the output

@returns                If the tests failed (based on the criteria)
"""
return_value, errors = ComplianceChecker.run_checker(path,
                                                     checker_names,
                                                     verbose,
                                                     criteria,
                                                     output_filename=output_filename,
                                                     output_format=output_format)

# Open the JSON output and get the compliance scores
with open(output_filename, 'r') as fp:
    cc_data = json.load(fp)
    scored = cc_data[cc_test[0]]['scored_points']
    possible = cc_data[cc_test[0]]['possible_points']
    log.debug('CC Scored {} out of {} possible points'.format(scored, possible))
```

## Compliance Checker Plug-Ins

Separate Plug-ins have been developed to complement the master Compliance Checker tool with
specifications for preparing data to be submitted to different data assembly centers.
The version numbering of these plug-ins are not necessarily link to the version of the
master Compliance Checker, but they are all designed to run with the master Compliance Checker tool.

