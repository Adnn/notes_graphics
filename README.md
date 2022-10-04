# Notes Graphics

## How To

### Files hosting

Files are currently hosted on Dropbox (avoiding duplication).
It is possible to restrict the dropbox sharing link to active user only:

    "Dropbox sharing" -> "Parameters" -> "Invited"

Yet, this forces to use the embedded Dropbox viewer, thus preventing the advanced parameters below:
* `?dl=1` will force browser download.
* replacing `www` subdomain  with `dl` might also [have an impact](https://www.dropboxforum.com/t5/Create-upload-and-share/public-links-to-raw-files/m-p/110392/highlight/true#M5978).
* `raw=1` will redirect to a link to the file (which opens pdf in browser direclty).
  * Copying the redirected link and appending `#page=30` would go to page 30 upon opening. [See this post](https://www.dropboxforum.com/t5/View-download-and-export/Navigate-to-a-particular-page-of-pdf-file-on-load/m-p/623265/highlight/true#M38854).
