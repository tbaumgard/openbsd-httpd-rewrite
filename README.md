# OpenBSD `httpd` Rewrite Patches

## OpenBSD 6.6 and newer

As of version 6.6, OpenBSD includes support for rewrites and has also [incorporated the supplemental patches](https://marc.info/?l=openbsd-cvs&m=155735202905914&w=2) found here. No patching is necessary.

## OpenBSD 6.5 and earlier

These patches add or enhance rewrite support in various versions of [OpenBSD's](https://www.openbsd.org/) [httpd](https://man.openbsd.org/httpd) web server.

### Applying a Patch

Follow the instructions in the [OpenBSD FAQ](https://www.openbsd.org/faq/faq5.html) for getting the source and any errata patches for your system.

After that, fetch and apply the corresponding rewrite patch, making sure to use the correct version for your system. The following example assumes that you're running 6.5:

```sh
cd /usr/src/usr.sbin/httpd
ftp 'https://raw.githubusercontent.com/tbaumgard/openbsd-httpd-rewrite/master/openbsd-httpd-rewrite-6.5.patch'
patch -p0 < openbsd-httpd-rewrite-6.5.patch
```

Once that's done, either build and install the entire system if it's outdated or do the following to build and install `httpd` only:

```sh
cd /usr/src/usr.sbin/httpd
make obj
make depend
make
make install
```

### OpenBSD 6.4 and 6.5

`httpd` includes support for URL rewrites as of OpenBSD 6.4. However, [there's a bug](https://marc.info/?l=openbsd-tech&m=153303654230606) that causes the `REQUEST_URI` CGI variable to be set to the rewritten URL instead of the requested URL. The patches for these versions fix this issue.

Additionally, the `$QUERY_STRING` macro was changed as of OpenBSD 6.4 so that it is no longer URL encoded. The patches add an additional macro, `$QUERY_STRING_ENC`, to continue to allow the option of using a URL-encoded query string.

The following partial configuration for [`httpd.conf`](https://man.openbsd.org/httpd.conf) demonstrates some common use cases.

```
server "www.example.com" {
	listen on egress port 80
	root "/www.example.com"

	# Let httpd handle assets like images, CSS, etc. directly.
	location "/assets/*" {
		pass
	}

	# Rewrite the old path for images directory.
	location match "^/images/(.*)" {
		request rewrite "/assets/images/%1"
	}

	# The setting above could be accomplished without rewrites like so:
	#location "/images/*" {
	#	root "/www.example.com/assets"
	#	pass
	#}

	# URL routing with query string modifications for a legacy script.
	# All predefined macros are enumerated and described in the httpd.conf(5)
	# man page, including the $QUERY_STRING_ENC added by the patch.
	location match "^/legacy/(.*)" {
		fastcgi socket "/run/php-fpm.sock"
		request rewrite "/legacy.php?target=%1&query=$QUERY_STRING_ENC"
	}

	# Default URL routing for everything else. The query string is untouched.
	location "*" {
		fastcgi socket "/run/php-fpm.sock"
		request rewrite "/default.php"
	}
}
```

### OpenBSD 5.9-6.3

`httpd` didn't include any support for URL rewrites prior to OpenBSD 6.4, so the patches for these versions add it.

An important distinction between the mechanism added in the patches and other common rewrite mechanisms available for Apache, nginx, etc. is that this one doesn't include the ability to rewrite URLs that have already been rewritten. This is to retain the same semantics of the `block return code uri` option and to keep things simple in terms of implementation and configuration.

The following partial configuration for [`httpd.conf`](https://man.openbsd.org/httpd.conf) demonstrates some common use cases.

```
server "www.example.com" {
	listen on egress port 80
	root "/www.example.com"

	# Let httpd handle assets like images, CSS, etc. directly.
	location "/assets/*" {
		pass
	}

	# Rewrite the old path for images directory.
	location match "^/images/(.*)" {
		pass rewrite "/assets/images/%1"
	}

	# The setting above could be accomplished without rewrites like so:
	#location "/images/*" {
	#	root "/www.example.com/assets"
	#	pass
	#}

	# URL routing with query string modifications for a legacy script.
	# All predefined macros are enumerated and described in the httpd.conf(5)
	# man page.
	location match "^/legacy/(.*)" {
		fastcgi socket "/run/php-fpm.sock"
		pass rewrite "/legacy.php?target=%1&query=$QUERY_STRING"
	}

	# Default URL routing for everything else. The query string is untouched.
	location "*" {
		fastcgi socket "/run/php-fpm.sock"
		pass rewrite "/default.php"
	}
}
```

## License

This work is available under a BSD license as described in the LICENSE.txt file.
