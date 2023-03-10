<p align="center">
  <img  src="./logo/crawlector_logo.svg">
</p>

# Crawl*e*ctor

Crawlector (the name Crawlector is a combination of **Crawl***er* & *Det***ector**) is a threat hunting framework designed for scanning websites for malicious objects.

**Note-1**: The framework was first presented at the [No Hat](https://www.nohat.it/2022/talks) conference in Bergamo, Italy on October 22nd, 2022 ([Slides](https://www.nohat.it/2022/static/slides/crawlector.pdf), [YouTube Recording](https://youtu.be/-9bupVXHo5Y)). Also, it was presented for the second time at the [AVAR](https://aavar.org/cybersecurity-conference/index.php/crawlector-a-threat-hunting-framework/) conference, in Singapore, on December 2nd, 2022.

**Note-2**: The accompanying tool [EKFiddle2Yara](https://github.com/MFMokbel/EKFiddle2Yara) (*is a tool that takes EKFiddle rules and converts them into Yara rules*) mentioned in the talk, was also released at both conferences.

# Features

- Supports spidering websites for findings additional links for scanning (up to 2 levels only)
- Integrates Yara as a backend engine for rule scanning
- Supports online and offline scanning
- Supports crawling for domains/sites digital certificate
- Supports querying URLhaus for finding malicious URLs on the page
- Supports hashing the page's content with [TLSH (Trend Micro Locality Sensitive Hash)](https://github.com/trendmicro/tlsh), and other standard cryptographic hash functions such as md5, sha1, sha256, and ripemd128, among others
  - TLSH won't return a value if the page size is less than 50 bytes or not "enough amount of randomness" is present in the data
- Supports querying the rating and category of every URL
- Supports expanding on a given site, by attempting to find all available TLDs and/or subdomains for the same domain
  - This feature uses the [Omnisint Labs](https://omnisint.io/) API (this site is down as of March 10, 2023) and RapidAPI APIs
  - TLD expansion implementation is native
  - This feature along with the rating and categorization, provides the capability to find scam/phishing/malicious domains for the original domain
- Supports domain resolution (IPv4 and IPv6)
- Saves scanned websites pages for later scanning (can be saved as a zip compressed)
- The entirety of the framework’s settings is controlled via a single customizable configuration file
- All scanning sessions are saved into a well-structured CSV file with a plethora of information about the website being scanned, in addition to information about the Yara rules that have triggered
- All HTTP(S) communications are proxy-aware
- One executable
- Written in C++

# URLHaus Scanning & API Integration

This is for checking for [malicious urls](https://urlhaus.abuse.ch/downloads/text/) against every page being scanned. The framework could either query the list of malicious URLs from URLHaus [server](https://urlhaus.abuse.ch/downloads/text/) (*configuration*: url_list_web), or from a file on disk (*configuration*: url_list_file), and if the latter is specified, then, it takes precedence over the former.

It works by searching the content of every page against all URL entries in url_list_web or url_list_file, checking for all occurrences. Additionally, upon a match, and if the configuration option check_url_api is set to true, Crawlector will send a POST request to the API URL set in the url_api configuration option, which returns a JSON object with extra information about a matching URL. Such information includes urlh_status (ex., online, offline, unknown), urlh_threat (ex., malware_download), urlh_tags (ex., elf, Mozi), and urlh_reference (ex., https://urlhaus.abuse.ch/url/1116455/). This information will be included in the log file cl_mlog_<*current_date*>_<*current_time*>_<(pm|am)>.csv (check below), only if check_url_api is set to true. Otherwise, the log file will include the columns urlh_url (list of matching malicious URLs) and urlh_hit (number of occurrences for every matching malicious URL), conditional on whether check_url is set to true.

URLHaus feature could be disabled in its entirety by setting the configuration option check_url to false.

It is important to note that this feature could slow scanning considering the huge number of [malicious urls](https://urlhaus.abuse.ch/downloads/text/) (~ 130 million entries at the time of this writing) that need to be checked, and the time it takes to get extra information from the URLHaus server (if the option check_url_api is set to true).

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

- Depending on the configuration settings (`log_to_file` or `log_to_cons`), if a Yara rule references only a module's attributes (ex., PE, ELF, Hash, etc...), then Crawlector will display only the rule's name upon a match, excluding offset and length data.

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
 - A value of 1, indicates a depth of level 2, with the value that comes after the “->” representing the maximum number of URLs to find, for every URL found per the **total** parameter. For clarification, and as shown in the example above, first, the spider will look for 10 URLs (as specified in the **total** parameter), and then, each of those found URLs will be spidered up to a max of 3 URLs; therefore, and in the best-case scenario, we would end up with `40 (10 + (10*3))` URLs.
 - The **sleep** parameter takes an integer value representing the number of milliseconds to sleep between every HTTP request.
 
**Note 1**: Type 3 URL could be turned into type 1 URL by setting the configuration parameter live_crawler to false, in the configuration file, in the spider section.

**Note 2**: Empty lines and lines that start with “;” or “//” are ignored.

# The Spider Functionality

The spider functionality is what gives Crawlector the capability to find additional links on the targeted page. The Spider supports the following featuers:

- The domain has to be of `Type 3`, for the Spider functionality to work
- You may specify a list of wildcarded patterns (pipe delimited) to prevent spidering matching urls via the `exclude_url` config. option. For example, `*.zip|*.exe|*.rar|*.zip|*.7z|*.pdf|.*bat|*.db`
- You may specify a list of wildcarded patterns (pipe delimited) to spider only urls that match the pattern via the `include_url` config. option. For example, `*/checkout/*|*/products/*`
- You may exclude HTTPS urls via the config. option `exclude_https`
- You may account for outbound/external links as well, for the main page only, via the config. option `add_ext_links`. This feature honours the `exclude_url` and `include_url` config. option.
- You may account for outbound/external links of the main page only, excluding all other urls, via the config. option `ext_links_only`. This feature honours the `exclude_url` and `include_url` config. option.

# Site Ranking Funcitonality

- This is for checking the ranking of the website
- You give it a file with a list of websites, with their ranking, in a csv file format
- Services that provide lists of websites ranking include, Alexa top-1m (discontinued as of May 2022), [Cisco Umbrella](https://umbrella-static.s3-us-west-1.amazonaws.com/index.html), [Majestic](https://majestic.com/reports/majestic-million), Quantcast, Farsight and [Tranco](https://tranco-list.eu/), among others
- CSV file format (2 columns only): first column holds the ranking, and the second column holds the domain name
- If a cell to contain quoted data, it'll be automatically dequoted
- Line breaks aren't allowed in quoted text
- Leading and trailing spaces are trimmed from cells read
- Empty and comment lines are skipped
- The section `site_ranking` in the configuration file provides some options to alter how the CSV file is to be read
- The performance of this query is dependent on the number of records in the CSV file
- Crawlector compares every entry in the CSV file against the domain being investigated, and not the other way around
- Only the registered/pay-level domain is compared

# Finding TLDs and Subdomains - [site] Section

- The `site` section provides the capability to expand on a given site, by attempting to find all available top-level domains (TLDs) and/or subdomains for the same domain. If found, new tlds/subdomains will be checked like any other domain
- This feature uses the Omnisint Labs (https://omnisint.io/) and RapidAPI APIs
- Omnisint Labs API returns subdomains and tlds, whereas RapidAPI returns only subdomains (the Omnisint Labs API is down as of March 10, 2023, however, the implementation is still available in case the site is back up)
- For RapidAPI, you need a valid "Domains records" API key that you can request from RapidAPI, and plug it into the key `rapid_api_key` in the configuration file
- With `find_tlds` enabled, in addition to Omnisint Labs API tlds results, the framework attempts to find other active/registered domains by going through every tld entry, either, in the `tlds_file` or `tlds_url`
- If tlds_url is set, it should point to a url that hosts tlds, each one on a new line (lines that start with either of the characters ';', '#' or '//' are ignored)
- `tlds_file`, holds the filename that contains the list of tlds (same as for `tlds_url`; only the tld is present, excluding the '.', for ex., "com", "org")
- If `tlds_file` is set, it takes precedence over `tlds_url`
- `tld_dl_time_out`, this is for setting the maximum timeout for the dnslookup function when attempting to check if the domain in question resolves or not
- `tld_use_connect`, this option enables the functionality to connect to the domain in question over a list of ports, defined in the option `tlds_connect_ports`
- The option `tlds_connect_ports` accepts a list of ports, comma separated, or a list of ranges, such as 25-40,90-100,80,443,8443 (range start and end are inclusive)
  - `tld_con_time_out`, this is for setting the maximum timeout for the connect function
- `tld_con_use_ssl`, enable/disable the use of ssl when attempting to connect to the domain
- If `save_to_file_subd` is set to true, discovered subdomains will be saved to "\expanded\exp_subdomain_<date>_<time>_<pm|am>.txt"
- If save_to_file_tld is set to true, discovered domains will be saved to "\expanded\exp_tld_<date>_<time>_<pm|am>.txt"
- If exit_here is set to true, then Crawlector bails out after executing this [site] function, irrespective of other enabled options

# Design Considerations

- A URL page is retrieved by sending a GET request to the server, reading the server response body, and passing it to Yara engine for detection.
- Some of the GET request attributes are defined in the [default] section in the configuration file, including, the User-Agent and Referer headers, and connection timeout, among other options.
- Although Crawlector logs a session's data to a CSV file, converting it to an SQL file is recommended for better performance, manipulation and retrieval of the data. This becomes evident when you’re crawling thousands of domains.
- Repeated domains/urls in the `cl_sites` are allowed.

# Limitations

-	Single threaded
-	Static detection (no dynamic evaluation of a given page's content)
- No headless browser support, yet!

# Third-party libraries used

- [Chilkat: library for website spidering, HTTP communications, hashing, JSON parsing and file compression (ZIP), among others](https://www.chilkatsoft.com/)
- [Yara: for rule scanning (v4.2.3)](https://github.com/virustotal/yara)
- [CrossGuid: for generating GUID/UUID](https://github.com/graeme-hill/crossguid)
- [Inih: for parsing configuration file](https://github.com/benhoyt/inih)
- [Rapidcsv: for parsing CSV files](https://github.com/d99kris/rapidcsv)
- [Color Console: for console coloring](https://github.com/imfl/color-console)
- [TLSH (Trend Micro Locality Sensitive Hash) (v4.8.2)](https://github.com/trendmicro/tlsh)

# Contributing

Open for pull requests and issues. Comments and suggestions are greatly appreciated.

# Author

Mohamad Mokbel ([@MFMokbel](https://twitter.com/MFMokbel))
