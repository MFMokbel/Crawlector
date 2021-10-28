<p align="center">
  <img  src="./logo/crawlector_logo.svg">
</p>

# Crawl*e*ctor
Crawlector (the name Crawlector is a combination of **Crawl***er* & *Det***ector**) is a threat hunting framework designed for scanning websites for malicious objects.

# Features
- Supports spidering websites for findings additional links for scanning (up to 2 levels only)
- Integrates Yara as a backend engine for rule scanning
- Supports online and offline scanning
- Supports crawling for domains/sites digital certificate
- Supports querying URLhaus for finding malicious URLs on the page
- Supports querying the rating and category of every URL
- Saves scanned websites pages for later scanning (can be saved as a zip compressed)
- The entirety of the framework’s settings is controlled via a single customizable configuration file
- All scanning sessions are saved into a well-structured CSV file with a plethora of information about the website being scanned, in addition to information about the Yara rules that have triggered
- One executable
- Written in C++

# URLHaus Scanning & API Integration
This is for checking for [malicious urls](https://urlhaus.abuse.ch/downloads/text/) against every page being scanned. The framework could either query the list of malicious URLs from URLHaus [server](https://urlhaus.abuse.ch/downloads/text/) (*configuration*: url_list_web), or from a file on disk (*configuration*: url_list_file), and if the latter is specified, then, it takes precedence over the former.

It works by searching the content of every page against all URL entries in url_list_web or url_list_file, checking for all occurrences. Additionally, upon a match, and if the configuration option check_url_api is set to true, Crawlector will send a POST request to the API URL set in the url_api configuration option, which returns a JSON object with extra information about a matching URL. Such information includes urlh_status (ex., online, offline, unknown), urlh_threat (ex., malware_download), urlh_tags (ex., elf, Mozi), and urlh_reference (ex., https://urlhaus.abuse.ch/url/1116455/). This information will be included in the log file cl_mlog_<*current_date*>_<*current_time*>_<(pm|am)>.csv (check below), only if check_url_api is set to true. Otherwise, the log file will include the columns urlh_url (list of matching malicious URLs) and urlh_hit (number of occurrences for every matching malicious URL), conditional on whether check_url is set to true.

URLHaus feature could be disabled in its entirety by setting the configuration option check_url to false.

It is important to note that this feature could slow scanning considering the huge number of [malicious urls](https://urlhaus.abuse.ch/downloads/text/) (~ 110 million entries at the time of this writing) that need to be checked, and the time it takes to get extra information from the URLHaus server (if the option check_url_api is set to true).

# Files and Folders Structures
1. \cl_sites
    + this is where the list of sites to be visited or crawled is stored.
    + supports multiple files and directories.
2. \crawled
    + where all crawled/spidered URLs are saved to a text file.
3. \certs
    + where all domains/sites digital certificates are stored (in .der format).
4. \results
    + where visited websites are saved.
5. \pg_cache
    + program cache for sites that are not part of the spider functionality.
6. \cl_cache
    + crawler cache for sites that are part of the spider functionality.
7. \yara_rules
    + this is where all Yara rules are stored. All rules that exist in this directory will be loaded by the engine, parsed, validated, and evaluated before execution.
8. cl_config.ini
    + this file contains all the configuration parameters that can be adjusted to influence the behavior of the framework.
9. cl_mlog_<*current_date*>_<*current_time*>_<(pm|am)>.csv
    + log file that contains a plethora of information about visited websites
    + date, time, the status of Yara scanning, list of fired Yara rules with the offsets and lengths of each of the matches, id, URL, HTTP status code, connection status, HTTP headers, page size, the path to a saved page on disk, and other columns related to URLHaus results.
    + file name is unique per session.
10. cl_offl_mlog_<*current_date*>_<*current_time*>_<(pm|am)>.csv
    + log file that contains information about files scanned offline.
    + list of fired Yara rules with the offsets and lengths of the matches, and path to a saved page on disk.
    + file name is unique per session.
11. cl_certs_<*current_date*>_<*current_time*>_<(pm|am)>.csv
    + log file that contains a plethora of information about found digital certificates

# Configuration File (cl_config.ini)

It is very important that you familiarize yourself with the configuration file cl_config.ini before running any session. All of the sections and parameters are documented in the configuration file itself.

The Yara offline scanning feature is a standalone option, meaning, if enabled, Crawlector will execute this feature only irrespective of other enabled features. And, the same is true for the crawling for domains/sites digital certificate feature. Either way, it is recommended that you disable all non-used features in the configuration file.

# Sites Format Pattern

To visit/scan a website, the list of URLs must be stored in text files, in the directory “cl_sites”. 

Crawlector accepts three types of URLs:

1. Type 1: one URL per line
    + Crawlector will assign a unique name to every URL, derived from the URL hostname
2. Type 2: one URL per line, with a unique name
 `[a-zA-Z0-9_-]{1,128} = <url>`
3. Type 3: for the spider functionality, a unique format is used. One URL per line is as follows:

 `<id>[`**depth**`:<0|1>-><\d+>,`**total**`:<\d+>,`**sleep**`:<\d+>] = <url>`

For example,

 `mfmokbel[depth:1->3,total:10,sleep:0] = https://www.mfmokbel.com`

which is equivalent to:
 `mfmokbel[d:1->3,t:10,s:0] = https://www.mfmokbel.com`

where, `<id> := [a-zA-Z0-9_-]{1,128}`

**depth**, **total** and **sleep**, can also be replaced with their shortened versions **d**, **t** and **s**, respectively.

- **depth**: the spider supports going two levels deep for finding additional URLs (this is a design decision).
 - A value of 0 indicates a depth of level 1, with the value that comes after the “->” ignored. 
 - A depth of level-1 is controlled by the total parameter. So, first, the spider tries to find that many additional URLs off of the specified URL.
 - The value after the “->” represents the maximum number of URLs to spider for each of the URLs found (as per the **total** parameter value).
 - A value of 1, indicates a depth of level 2, with the value that comes after the “->” representing the maximum number of URLs to find, for every URL found (depth-0) per the **total** parameter. For clarification, and as shown in the example above, first, the spider will look for 10 URLs (as specified in the **total** parameter), and then, each of those found URLs will be spidered up to a max of 3 URLs; therefore, and in the best-case scenario, we would end up with `40 (10 + (10*3))` URLs.
 - The **sleep** parameter takes an integer value representing the number of milliseconds to sleep between every HTTP request.
 
**Note 1**: Type 3 URL could be turned into type 1 URL by setting the configuration parameter live_crawler to false, in the configuration file, in the spider section.

**Note 2**: Empty lines and lines that start with “;” or “//” are ignored.

# Limitations
-	Single threaded
-	Static detection

# Third-party libraries used

- [Chilkat: library for website spidering, HTTP communications, JSON parsing, and file compression (ZIP)](https://www.chilkatsoft.com/)
- [Yara: for rule scanning (v4.1.1)](https://github.com/virustotal/yara)
- [CrossGuid: for generating GUID/UUID](https://github.com/graeme-hill/crossguid)
- [Inih: for parsing configuration file](https://github.com/benhoyt/inih)
- [Color Console: for console coloring](https://github.com/imfl/color-console)

# Contributing

Open for pull requests and issues. Comments and suggestions are greatly appreciated.

# Author

Mohamad Mokbel ([@MFMokbel](https://twitter.com/MFMokbel))
