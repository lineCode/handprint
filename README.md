Handprint<img width="100px" align="right" src=".graphics/noun_Hand_733265.svg">
=========

An experiment with handwritten text optical recognition on Caltech Archives materials.

*Authors*:      [Michael Hucka](http://github.com/mhucka)<br>
*Repository*:   [https://github.com/caltechlibrary/handprint](https://github.com/caltechlibrary/handprint)<br>
*License*:      BSD/MIT derivative &ndash; see the [LICENSE](LICENSE) file for more information

[![License](https://img.shields.io/badge/License-BSD%203--Clause-blue.svg?style=flat-square)](https://choosealicense.com/licenses/bsd-3-clause)
[![Python](https://img.shields.io/badge/Python-3.4+-brightgreen.svg?style=flat-square)](http://shields.io)
[![Latest release](https://img.shields.io/badge/Latest_release-0.3.0-b44e88.svg?style=flat-square)](http://shields.io)

Table of Contents
-----------------

* [Introduction](#-introduction)
* [Installation instructions](#-installation-instructions)
   * [Install Handprint on your computer](#-install-handprint-on-your-computer)
   * [Obtain cloud service credentials](#-obtain-cloud-service-credentials)
      * [<em>Microsoft</em>](#microsoft)
      * [<em>Google</em>](#google)
* [Running Handprint](#︎-running-handprint)
   * [File formats recognized](#file-formats-recognized)
   * [Supported HTR/OCR methods](#supported-htrocr-methods)
   * [Service account credentials](#service-account-credentials)
   * [Files versus URLs](#files-versus-urls)
   * [Command line options](#command-line-options)
* [Data returned](#︎-data-returned)
* [Getting help and support](#-getting-help-and-support)
* [Acknowledgments](#︎-acknowledgments)
* [Copyright and license](#︎-copyright-and-license)

☀ Introduction
-----------------------------

Handprint (_**Hand**written **P**age **R**ecognit**i**o**n** **T**est_) is a small project to examine the use of alternative optical character recognition (OCR) and handwritten text recognition (HTR) methods on documents from the [Caltech Archives](http://archives.caltech.edu).  Tests include the use of Google's OCR/HTR capabilities in their [Google Cloud Vision API](https://cloud.google.com/vision/docs/ocr) and [Tesseract](https://en.wikipedia.org/wiki/Tesseract_(software)).

✺ Installation instructions
---------------------------

Handprint is a program written in Python 3 that works by invoking cloud-based services.  Installation requires both obtaining a copy of Handprint itself, and also signing up for access to the cloud service providers.

### ⓵&nbsp;&nbsp; _Install Handprint on your computer_

The following is probably the simplest and most direct way to install this software on your computer:
```sh
sudo pip3 install git+https://github.com/caltechlibrary/handprint.git --upgrade
```

Alternatively, you can instead clone this GitHub repository and then run `setup.py` manually.  First, create a directory somewhere on your computer where you want to store the files, and cd to it from a terminal shell.  Next, execute the following commands:
```sh
git clone https://github.com/caltechlibrary/handprint.git
cd handprint
sudo python3 -m pip install . --upgrade
```

### ⓶&nbsp;&nbsp; _Obtain cloud service credentials_

Credentials for different services need to be provided to Handprint in the form of JSON files.  Each service needs a separate JSON file named after the service (e.g., `microsoft.json`) and placed in a directory that Handprint searches.  By default, Handprint searches for the files in a subdirectory named `creds` where Handprint is installed, but an alternative diretory can be indicated at run-time using the `-c` command-line option (or `/c` on Windows).

The specific contents and forms of the files differ depending on the particular service, as described below.

### _Microsoft_

Microsoft's approach to credentials in Azure involves the use of [subscription keys](https://docs.microsoft.com/en-us/azure/cognitive-services/computer-vision/vision-api-how-to-topics/howtosubscribe).  The credentials file for Handprint just needs to contain a single field:

```json
{
 "subscription_key": "YOURKEYHERE"
}
```

The value of "YOURKEYHERE" will be a string such as `"18de248475134eb49ae4a4e94b93461c"`.  To sign up for Azure and obtain a key, visit [https://portal.azure.com](https://portal.azure.com) and sign in using your Caltech Access email address/login.  (Note: you will need to turn off browser security plugins such as Ad&nbsp;Block and uMatrix if you have them, or else the site will not work.)  It will redirect you to the regular Caltech Access login page and then (after you log in) back to the Dashboard [https://portal.azure.com](https://portal.azure.com), from where you can create credentials.  Some notes about this can be found in the [project Wiki pages](https://github.com/caltechlibrary/handprint/wiki/Getting-Microsoft-Azure-credentials).

When signing up for an Azure cloud service account, make sure to choose "Western US" as the region so that the service URL begins with "https://westus.api.cognitive.microsoft.com".

### _Google_

Credentials for using a Google service account are stored in a JSON file containing many fields.  The overall form looks like this:

```
{
  "type": "service_account",
  "project_id": "theid",
  "private_key_id": "thekey",
  "private_key": "-----BEGIN PRIVATE KEY-----anotherkey-----END PRIVATE KEY-----\n",
  "client_email": "emailaddress",
  "client_id": "id",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://oauth2.googleapis.com/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "someurl"
}
```

Getting one of these files is unfortunately a complicated process.  It's summarized in the Google Cloud documentation for [Creating a service account](https://cloud.google.com/docs/authentication/), but some more explicit instructions can be found in our Handprint [project Wiki pages](https://github.com/caltechlibrary/handprint/wiki/Getting-Google-Cloud-credentials).


▶︎ Running Handprint
------------------

Handprint is a command-line driven program.  There is a single command-line interface program called `handprint`.  You can run it by starting a terminal shell and `cd`'ing to the directory where you installed Handprint, and then running the program `bin/handprint` from there.  For example:

```bash
bin/handprint -h
```

Alternatively, you should be able to run Handprint from anywhere using the normal approach to running Python modules:

```bash
python3 -m handprint -h
```

The `-h` option (`/h` on Windows) will make `handprint` display some help information and exit immediately.  To make Handprint do more, you can supply other arguments that instruct Handprint to process image files (or alternatively, URLs pointing to image files at a network location) and run handwritten text recognition (HTR) or optical character recognition (OCR) algorithms on them, as explained below.


### File formats recognized

Whether the images are stored locally or accessible via URLs, each image should be a single page of a document in which text should be recognized.  The accepted by the cloud services at this time are JPEG, PNG, GIF, and BMP only, but Handprint can convert a few others formats into JPEG if necessary.  Specifically, Handprint also handles JPEG 2000 and TIFF formats, which it converts to JPEG before sending to the different methods for text recognition.


### Supported HTR/OCR methods

Handprint can contact more than one cloud service for OCR and HTR.  You can use the `-l` option (`/l` on Windows) to make Handprint display a list of the methods currently implemented:

```
# bin/handprint -l
Known methods (for use as values for option -m):
   microsoft
   google
```

By default, Handprint will run each known method in turn.  To invoke only one specific method, use the `-m` option (`/m` on Windows) followed by a method name:

```bash
bin/handprint -m microsoft /path/to/images
```


### Service account credentials

Handprint looks for credentials files in the directory where it is installed, but you can put credentials in another directory and then tell Handprint where to find it using the `-c` option (`/c` on Windows).  Example of use:

```bash
bin/handprint -c ~/handprint-credentials /path/to/images
```


### Files versus URLs

Handprint can work both with files and with URLs.  By default, arguments are interpreted as being files or directories of files, but if given the `-u` option (`/u` on Windows), the arguments are interpreted instead as URLs pointing to images.

A challenge with using URLs is how to name the files that Handprint writes for the results.  Some CMS systems store content using opaque schemes that provide no clear names in the URLs, making it impossible for a software tool such as Handprint to guess what file name would make sense to use for local storage.  Worse, some systems create extremely long URLs, making it impractical to use the full URL itself as the file name.  For example, the following is a real URL pointing to an image in Caltech Archives today:

```
https://hale.archives.caltech.edu/adore-djatoka//resolver?rft_id=https%3A%2F%2Fhale.archives.caltech.edu%2Fislandora%2Fobject%2Fhale%253A85240%2Fdatastream%2FJP2%2Fview%3Ftoken%3D7997253eb6195d89b2615e8fa60708a97204a4cdefe527a5ab593395ac7d4327&url_ver=Z39.88-2004&svc_id=info%3Alanl-repo%2Fsvc%2FgetRegion&svc_val_fmt=info%3Aofi%2Ffmt%3Akev%3Amtx%3Ajpeg2000&svc.format=image%2Fjpeg&svc.level=4&svc.rotate=0
```

To deal with this situation, Handprint manufactures its own file names when the `-u` option is used.  The scheme is simple: by default, Handprint will use a base name of `document-N`, where `N` is an integer.  The integers start from `1` for every run of Handprint, and the integers count the URLs found either on the command line or in the file indicated by the `-f` option.  The image found at a given URL is stored in a file named `document-N.E` where `E` is the format extension (e.g., `document-1.jpeg`, `document-1.png`, etc.).  The URL itself is stored in another file named `document-1.url`.  Thus, the files produced by Handprint will look like this when the `-u` option is used:

```
document-1.jpeg
document-1.url
document-1.google.txt
document-1.google.json
document-1.microsoft.txt
document-1.microsoft.json

document-2.jpeg
document-2.url
document-2.google.txt
document-2.google.json
document-2.microsoft.txt
document-2.microsoft.json

document-3.jpeg
document-3.url
document-3.google.txt
document-3.google.json
document-3.microsoft.txt
document-3.microsoft.json

...
```

The base name `image` can be changed using the `-r` option (`/r` on Windows).  For example, running Handprint with the option `-r einstein` will cause the outputs to be named `einstein-1.jpeg`, `einstein-1.url`, etc. (assuming, for the sake of this example, that the image file format is `jpeg`).

The use of the `-u` option also **requires the use of the `-o` option** (`/o` on Windows) to tell Handprint where to store the results.  This is a consequence of the fact that, without being provided with files or directories on the local disk, Handprint can't infer where to write its output.

Example of use:

```bash
bin/handprint -u -f /tmp/urls-to-read.txt -o /tmp/results/
```

Finally, note that providing URLs on the command line can be problematic due to how terminal shells interpret certain characters, and so when supplying URLs, it's usually better to list the URLs in a file in combination with the `-f` option (`/f` on Windows).


### Command line options

The following table summarizes all the command line options available. (Note: on Windows computers, `/` must be usedas the prefix character instead of `-`):

| Short    | Long&nbsp;form&nbsp;opt | Meaning | Default |  |
|----------|-------------------|----------------------|---------|---|
| `-c`_D_  | `--creds-dir`_D_  | Look for credentials in directory _D_ | `creds` |
| `-f`_F_  | `--from-file`_F_  | Read file names or URLs from file _F_ | Use names or URLs given on command line |
| `-l`     | `--list`          | Disply list of known methods | |
| `-m`_M_  | `--method`_M_     | Use method _M_ | "all" |
| `-o`_O_  | `--output`_O_     | Write outputs to directory _D_ | Same directories where images are found |  ⚑ |
| `-u`     | `--given-urls`    | Inputs are URLs, not files or dirs | Assume files and/or directories of files |
| `-r`_R_  | `--root-name`_R_  | Write outputs to files named _R_-n | Use the base names of the image files | ✦ |
| `-q`     | `--quiet`         | Don't print messages while working | Be chatty while working |
| `-C`     | `--no-color`      | Don't color-code the output | Use colors in the terminal output |
| `-D`     | `--debug`         | Debugging mode | Normal mode |
| `-V`     | `--version`       | Print program version info and exit | Do other actions instead |

 ⚑ &nbsp; The `o` option (`/o` on Windows) **must be provided** if the `-u` option (`/u` on Windows) is used: the results must be written to the local disk somewhere, because it is not possible to write the results in the network locations represented by the URLs.

✦ &nbsp; If `-u` is used (meaning, the inputs are URLs and not files or directories), then the outputs will be written by default to names of the form `document-n`, where n is an integer.  Examples: `document-1.jpeg`, `document-1.google.txt`, etc.  This is because images located in network content management systems may not have any clear names in their URLs.


⚛︎ Data returned
---------------

Handprint tries to gather all the data that each service returns for text recognition, and outputs the results in two forms: a `.json` file containing all the results, and a `.txt` file containing just the document text.  The exact content of the `.json` file differs for each service.


⁇ Getting help and support
--------------------------

If you find an issue, please submit it in [the GitHub issue tracker](https://github.com/caltechlibrary/handprint/issues) for this repository.


☺︎ Acknowledgments
-----------------------

The [vector artwork](https://thenounproject.com/search/?q=hand&i=733265) of a hand used as a logo for Handprint was created by [Kevin](https://thenounproject.com/kevn/) from the Noun Project.  It is licensed under the Creative Commons [CC-BY 3.0](https://creativecommons.org/licenses/by/3.0/) license.

Handprint makes use of numerous open-source packages, without which it would have been effectively impossible to develop Turf with the resources we had.  We want to acknowledge this debt.  In alphabetical order, the packages are:

* [colorama](https://github.com/tartley/colorama) &ndash; makes ANSI escape character sequences work under MS Windows terminals
* [google-api-core, google-api-python-client, google-auth, google-auth-httplib2, google-cloud, google-cloud-vision, googleapis-common-protos, google_api_python_client](https://github.com/googleapis/google-cloud-python) &ndash; Google API libraries 
* [halo](https://github.com/ManrajGrover/halo) &ndash; busy-spinners for Python command-line programs
* [httplib2](https://github.com/httplib2/httplib2) &ndash; a comprehensive HTTP client library
* [ipdb](https://github.com/gotcha/ipdb) &ndash; the IPython debugger
* [oauth2client](https://github.com/googleapis/oauth2client) &ndash; Google OAuth 2.0 library
* [plac](http://micheles.github.io/plac/) &ndash; a command line argument parser
* [requests](http://docs.python-requests.org) &ndash; an HTTP library for Python
* [setuptools](https://github.com/pypa/setuptools) &ndash; library for `setup.py`
* [termcolor](https://pypi.org/project/termcolor/) &ndash; ANSI color formatting for output in terminal

☮︎ Copyright and license
---------------------

Copyright (C) 2018, Caltech.  This software is freely distributed under a BSD/MIT type license.  Please see the [LICENSE](LICENSE) file for more information.
    
<div align="center">
  <a href="https://www.caltech.edu">
    <img width="100" height="100" src=".graphics/caltech-round.svg">
  </a>
</div>
