# OpenBSD `httpd` Rewrite Support

These patches add basic rewrite support to various versions of [OpenBSD's](https://www.openbsd.org/) [httpd](http://man.openbsd.org/OpenBSD-current/man8/httpd.8) web server without having to use redirects.

An important distinction between this mechanism and other common rewrite mechanisms available for Apache, nginx, etc. is that this one doesn't include the ability to rewrite URLs that have already been rewritten. This is to retain the same semantics of the `block return code uri` option and to keep things simple in terms of implementation and configuration.

## Applying a Patch

Follow the instructions in the [OpenBSD FAQ](http://www.openbsd.org/faq/faq10.html#Patches) for getting the source and any errata patches for your system.

After that, fetch and apply the rewrite patch, making sure to use the correct version for your system. The following example assumes that you're running -current:

```sh
cd /usr/src/usr.sbin/httpd
ftp 'https://raw.githubusercontent.com/tbaumgard/openbsd-httpd-rewrite/master/openbsd-httpd-rewrite-current.patch'
patch -p0 < openbsd-httpd-rewrite-current.patch
```

Once that's done, either build and install the entire system if it's outdated or do the following to build and install `httpd` only:

```sh
cd /usr/src/usr.sbin/httpd
make obj
make depend
make
make install
```

## Example Configuration

The following partial configuration for [`httpd.conf`](http://man.openbsd.org/OpenBSD-current/man5/httpd.conf.5) demonstrates some common use cases.

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
	#	root {
	#		"/www.example.com/assets/images"
	#		strip 1
	#	}
	#
	#	pass
	#}

	# URL routing with query string modifications for a legacy script.
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
