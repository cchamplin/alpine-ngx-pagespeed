Alpine Nginx Pagespeed
======================

by Caleb Champlin (@cchamplin) <caleb.champlin@gmail.com>

The files in this repo are unless otherwise mentioned MIT licensed.

About
-----

This APKBUILD and patch files build an Alpine package for the ngx_pagespeed module from Google.

The module is built as a dynamic nginx library and at a minimum requires nginx-1.9.11

Notes
-----

The package uses tarballs from:
https://github.com/pagespeed/mod_pagespeed/issues/968 (PSOL)
https://github.com/pagespeed/ngx_pagespeed/releases (ngx_pagespeed)

TODO
----

Right now libpng12 libraries are being distributed as part of this package, this should be fixed.


