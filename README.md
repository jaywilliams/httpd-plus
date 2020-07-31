# httpd-plus

Add-ons for the [OpenBSD](https://www.openbsd.org) [`httpd(8)`](http://man.openbsd.org/httpd) web server, always applicable to the lastest `-stable` and `-current` branches.

## List of add-ons

### updates

Bug fixes:
* Failing detection of `location` duplicates (see on [tech@](https://marc.info/?l=openbsd-tech&m=157313087000813))

Commits to `-current` merged into `-stable`:
* [Commits on May 16, 2020](https://github.com/openbsd/src/commit/83a0d42a38eeac687972cf5d09e1d9b2f003b4cf)
* [Commits on May 18, 2020](https://github.com/openbsd/src/commit/aa3aed545c710959415b79b9b939bd135e4ca9f6)
* [Commits on May 22, 2020](https://github.com/openbsd/src/commit/8643c0edbdaf122a89210ba0359ae06546ed9f17)
* [Commits on July 25, 2020](https://github.com/openbsd/src/commit/8ae2ff809e5f555ca12a854168511dd4bdbb8de2)
* [Commits on July 30, 2020](https://github.com/openbsd/src/commit/8e304376b367ee76f257ecd22fae486aa9f20325)

### cache-control-headers

Optional HTTP `Cache-Control` headers via `httpd.conf(5)`.

```
types {
	...
	image/jpeg  { cache "max-age=2592000, public" }             jpeg jpg
	text/css    { cache "max-age=86400, private" }              css
	text/html   { cache "no-store, no-cache, must-revalidate" } html
	...
}
```

### wordpress-pretty-permalinks

Access tests (`found` or `not found`) for `location` resource paths via `httpd.conf(5)`. This enables [WordPress](https://wordpress.org) [Pretty Permalinks](https://wordpress.org/support/article/using-permalinks/) just like on an Apache web server with `mod_rewrite` installed.

```
server "www.example.com" {
	listen on * port www
	directory index "index.php"

	location not found "/*" {
		request rewrite "/index.php"
	}
	location "/*.php" {
		fastcgi socket "/run/php-fpm.sock"
	}
}
```

_Note:_ Even with this add-on installed, WordPress is unable to discover that the OpenBSD web server is now capable to perform required URL rewrites. This will make the [Permalink Settings Screen](https://wordpress.org/support/article/settings-permalinks-screen/) not behave as expected. Luckily, and for this case exactly, the [got_url_rewrite hook](https://developer.wordpress.org/reference/hooks/got_url_rewrite/) exists. Adding the following line of code into the current theme's `functions.php` file will straighten things out.

```
add_filter('got_url_rewrite', '__return_true');
```

### fastcgi-script-overrides

Definition of `script` overrides for `location` specific `fastcgi` environments via `httpd.conf(5)`.

```
server "www.example.com" {
	...
	location "/foobar/*" {
		fastcgi {
			socket "/run/php-fpm.sock"
			script "/override.php"
		}
	}
	...
}
```

### client-ip-filters

Client IP matching (`from` or `not from`) for `location` sections in `httpd.conf(5)`.

```
server "www.example.com" {
	listen on * port www

	location "/intranet*" not from "10.0.0/24" { block }
	...
}
```

## How to install

`httpd-plus` is a series of patch files always applicable to the lastest `-stable` and `-current` branches. Just follow the steps below.

1. Become `root`.
1. Make sure the `/usr/src` tree is in place and up-to-date.
1. Get patch files and installation script downloaded and extracted.

	```
	# ftp -o - https://codeload.github.com/mpfr/httpd-plus/tar.gz/master | tar xzvf -
	httpd-plus-master
	httpd-plus-master/00-updates-current.patch
	httpd-plus-master/00-updates-stable.patch
	httpd-plus-master/01-cache-control-headers-current.patch
	httpd-plus-master/01-cache-control-headers-stable.patch
	httpd-plus-master/02-wordpress-pretty-permalinks-current.patch
	httpd-plus-master/02-wordpress-pretty-permalinks-stable.patch
	httpd-plus-master/03-fastcgi-script-overrides-current.patch
	httpd-plus-master/03-fastcgi-script-overrides-stable.patch
	httpd-plus-master/04-client-ip-filters-current.patch
	httpd-plus-master/04-client-ip-filters-stable.patch
	httpd-plus-master/README.md
	httpd-plus-master/install
	#
	```

1. Apply the patch files by running the installation script which will build and install the `httpd-plus` binary (`-stable` or `-current` branch will be detected automatically). After that, the original source code will be restored.

	```
	# sh httpd-plus-master/install
	Identified -current branch.
	Backing up original sources ... Done.
	Applying patch files ...
	... 00-updates-current ...
	Hmm...  Looks like a unified diff to me...
	The text leading up to this was:
	--------------------------
	|Index: usr.sbin/httpd/parse.y
	--------------------------
	Patching file usr.sbin/httpd/parse.y using Plan A...
	Hunk #1 succeeded at 564.
	done
	... 01-cache-control-headers-current ...
	Hmm...  Looks like a unified diff to me...
	.
	.
	.
	done
	Building and installing httpd-plus binary and manpage ...
	.
	.
	.
	Restoring original sources ... Done.

	Installing httpd-plus binary and manpage completed successfully.
	Please consult 'man httpd.conf' for further information on new features.
	#
	```

1. Adapt your `httpd.conf` to newly added features. For further information, just have a look at the updated `httpd.conf(5)` manpage via `man httpd.conf`. Make sure your new configuration is valid.

	```
	# httpd -n
	configuration OK
	#
	```

1. Restart the `httpd` daemon.

	```
	# rcctl restart httpd
	httpd(ok)
	httpd(ok)
	#
	```

## How to uninstall

As patching the source code will be undone automatically right after building and installing the `httpd-plus` daemon, the original version may be easily recovered by performing a de novo rebuild and reinstall.

```
# cd /usr/src/usr.sbin/httpd
# make obj
# make clean
# make
# make install
# rcctl restart httpd
```
