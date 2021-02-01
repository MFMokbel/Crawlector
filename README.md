# Crawlector
Crawlector (the name Crawlector is a combination of **Crawl**er & Det**ector**) is a threat hunting framework designed for scanning websites for malicious objects.

# Features
- Supports spidering websites for findings additional links for scanning (up to 2 levels only)
- Integrates Yara as a backend engine for rule scanning
- Supports online and offline scanning
- Saves scanned websites pages for later scanning (can be saved as zip compressed)
- The entirety of the framework’s settings is controlled via a single customizable configuration file
- All scanning sessions are saved into a well-structured csv file with plethora of information about the website being scanned, in addition to information about the Yara rules that have triggered
- One executable
- Written in C++

# Files and Folders Structures
1. \cl_sites
- this is where the list of sites to be visited or crawled is stored.
- supports multiple files and directories.
2. \crawled
- where all crawled/spidered urls are saved to a text file.
3. \results
- where visited websites are saved.
4. \pg_cache
- program cache for sites that are not part of the spider functionality.
5.\cl_cache
- crawler cache for sites that are part of the spider functionality.
6. \yara_rules
- this is where all Yara rules are stored. All rules that exist in this directory will be loaded by the engine, parsed, validated, and evaluated prior to execution.
7. cl_config.ini
- this file contains all the configuration parameters that can be adjusted to influence the behavior of the framework.
8. cl_mlog_<*current_date*>_<*current_time*>_<(pm|am)>.csv
- log file that contains plethora of information about visited websites
- date, time, list of fired Yara rules with the offsets and lengths of each of the matches, id, url, status code, connection status, HTTP headers, page size, and path to saved page on disk.
- file name is unique per session.
9. cl_offl_mlog_<*current_date*>_<*current_time*>_<(pm|am)>.csv
- log file that contains information about files scanned offline.
- list of fired Yara rules with the offsets and lengths of the matches, and path to saved page on disk.
b.	file name is unique per session.

**Note**: It is very important that you familiarize yourself with the configuration file cl_config.ini prior to running any session. All of the sections and parameters are documented in the configuration file itself.

# Sites Format Pattern

To visit/scan a website, the list of urls must be stored in text files, in the directory “cl_sites”. Crawlector accepts three types of urls:

1. Type 1: one url per line
- Crawlector will assign a unique name to every url, derived from the url host name
2. Type 2: one url per line, with a unique name
- `[a-zA-Z0-9_-]{1,128} = <url>`
3. Type 3: for the spider functionality, a unique format is used. One url per line as follows:

- `<id>[`**depth**`:<0|1>-><\d+>,`**total**`:<\d+>,`**sleep**`:<\d+>] = <url>`

For example,

`mfmokbel[depth:1->3,total:10,sleep:0] = https://www.mfmokbel.com`

which is equivalent to:
`mfmokbel[d:1->3,t:10,s:0] = https://www.mfmokbel.com`

where, `<id> := [a-zA-Z0-9_-]{1,128}`

depth, total and sleep, can also be replaced with their shortened versions d, t and s, respectively.

- depth; the spider supports going two-levels deep for finding additional urls (this is a design decision).
 - A value of 0, indicates a depth of level 1, with the value that comes after the “->” ignored. 
 - A depth of level-1 is controlled by the total parameter. So, first, the spider tries to find that many number of additional urls off of the specified url.
 - The value after the “->” represents the maximum number of urls to spider for each of the urls found (as per the total parameter value).
 - A value of 1, indicates a depth of level 2, with the value that comes after the “->” representing the maximum number of urls to find, for every url found (depth-0) per the total parameter. For clarification, and as shown in the example above, first, the spider will look for 10 urls (as specified in the total parameter), and then, each of those found urls will be spidered up to a max of 3 urls; therefore, and in the best-case scenario, we would end up with 40 (10 + (10*3)) urls.
 - The sleep parameter takes an integer value representing the number of milliseconds to sleep between every HTTP request.
 
**Note**: Type 3 url could be turned into type 1 url by setting the configuration parameter live_crawler to false, in the configuration file, in the spider section.
Empty lines and lines that start with “;” or “//” are ignored.

# Limitations
-	Single threaded
-	Static detection

# Third-party libraries used

- [Chilkat: library for website spidering, HTTP communications, and file compression (ZIP)](https://www.chilkatsoft.com/)
- [Yara: for rule scanning (v4.0.2)](https://github.com/virustotal/yara)
- [CrossGuid: for generating GUID/UUID](https://github.com/graeme-hill/crossguid)
- [Inih: for parsing configuration file](https://github.com/benhoyt/inih)
- [Color Console: for console coloring](https://github.com/imfl/color-console)

# Contributing

Open for pull requests and issues. Comments and suggestions are greatly appreciated.
