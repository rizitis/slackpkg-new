## slackpkg-unofficial

This is a **patched** version of slackpkg<br>
- support /etc/slackpkg/whitelist
- keeping support for slackpkgpluss

---

## HowTo

This slackpkg version has in /etc/slackpkg a whitelist file.<br>
Every package name listed there escape from set blacklist. <br>

For example assume for your personal reasons having in blacklist the  `kde/` set. <br>
But you want some packages to included in you installation from this set (or any other) and receive official updates.<br> 
Just open whitelist and name them. example: `cat /etc/slackpkg/whitelist`
```
qca
futuresql
qcoro
digikam
kactivities
kactivities-stats
kstars
```
All these packages will be 

## Terminal Demo

[![Watch the demo](assets/demo.png)](https://asciinema.org/a/zhEjSRZTx23JD52RY5eiS3yTR)

---


## NOTE:

This slackpkg version is a personal fork, **Slackware project or slackpkg has no responsibility**<br>

This is the patch:
```
diff -Naupr slackpkg/README.md slackpkg-new/README.md
--- slackpkg/README.md	2021-07-07 06:38:51.000000000 +0300
+++ slackpkg-new/README.md	2025-04-03 11:17:56.879228054 +0300
@@ -1,5 +1,46 @@
-Slackpkg is a tool for installing or upgrading packages through a network.
-You can make a minimal installation of Slackware Linux and then install 
-additional packages from a Slackware mirror.
+## slackpkg-unofficial
+
+This is a **patched** version of slackpkg<br>
+- support /etc/slackpkg/whitelist
+- keeping support for slackpkgpluss
+
+---
+
+## HowTo
+
+This slackpkg version has in /etc/slackpkg a whitelist file.<br>
+Every package name listed there escape from set blacklist. <br>
+
+For example assume for your personal reasons having in blacklist the  `kde/` set. <br>
+But you want some packages to included in you installation from this set (or any other) and receive official updates.<br> 
+Just open whitelist and name them. example: `cat /etc/slackpkg/whitelist`
+```
+qca
+futuresql
+qcoro
+digikam
+kactivities
+kactivities-stats
+kstars
+```
+All these packages will be 
+
+## Terminal Demo
+
+[![Watch the demo](assets/demo.png)](https://asciinema.org/a/zhEjSRZTx23JD52RY5eiS3yTR)
+
+---
+
+
+## NOTE:
+
+This slackpkg version is a personal fork, **Slackware project or slackpkg has no responsibility**<br>
+
+
+---
+
+## trademarks
+
+Slackware is a [trademark](http://www.slackware.com/trademark/trademark.php) of Patrick Volkerding. <br>
+©[Slackpkg](https://slackpkg.org/) Some Rights Reserved - Github [page](https://github.com/rworkman/slackpkg)
 
-Releases are available from [slackpkg.org](https://slackpkg.org/).
diff -Naupr slackpkg/doinst.sh slackpkg-new/doinst.sh
--- slackpkg/doinst.sh	2021-10-12 08:34:08.890488416 +0300
+++ slackpkg-new/doinst.sh	2025-04-03 10:36:50.136152775 +0300
@@ -39,6 +39,7 @@ copy_mirror_file
 config etc/slackpkg/mirrors.new
 config etc/slackpkg/slackpkg.conf.new
 config etc/slackpkg/blacklist.new
+config etc/slackpkg/whitelist.new
 rm -f var/lib/slackpkg/ChangeLog.txt
 rm -f var/lib/slackpkg/pkglist
 rm -f var/lib/slackpkg/CHECKSUMS.md5*
diff -Naupr slackpkg/files/core-functions.sh slackpkg-new/files/core-functions.sh
--- slackpkg/files/core-functions.sh	2024-07-20 20:06:41.996062136 +0300
+++ slackpkg-new/files/core-functions.sh	2025-04-03 11:27:40.596245867 +0300
@@ -628,38 +628,80 @@ function givepriority {
 # names (lpkg) and packages unique to one or other file (dpkg)
 #
 function listpkgname() {
-	cut -f2 -d\  ${TMPDIR}/pkglist | sort > ${TMPDIR}/spkg	
-	cut -f2 -d\  ${TMPDIR}/tmplist | sort > ${TMPDIR}/lpkg
-	cat ${TMPDIR}/pkglist ${TMPDIR}/tmplist | \
-		cut -f2-6 -d\ |sort | uniq -u | \
-		cut -f1 -d\  | uniq > ${TMPDIR}/dpkg
+    # Generate spkg (mirror package names) and lpkg (local package names)
+    cut -f2 -d\  ${TMPDIR}/pkglist | sort > ${TMPDIR}/spkg
+    cut -f2 -d\  ${TMPDIR}/tmplist | sort > ${TMPDIR}/lpkg
+
+    # Create dpkg (packages unique to one or the other file)
+    cat ${TMPDIR}/pkglist ${TMPDIR}/tmplist | \
+        cut -f2-6 -d\ | sort | uniq -u | \
+        cut -f1 -d\  | uniq | sort > ${TMPDIR}/dpkg
+
+    # Ensure dpkg is sorted for the comm comparison
+    sort ${TMPDIR}/dpkg -o ${TMPDIR}/dpkg
+
+    # Check if the whitelist exists and log its contents for debugging
+    if [ -f $CONF/whitelist ]; then
+        echo "Whitelist found, processing..."
+        # Process and append the whitelist to dpkg list
+      while read whitelist_package; do
+        # Skip lines that are comments (starting with #) or empty lines
+		if [[ "$whitelist_package" =~ ^#.* ]] || [[ -z "$whitelist_package" ]]; then
+        continue
+		fi
+
+            # Check if the package is already in dpkg
+            if grep -q "^$whitelist_package$" ${TMPDIR}/dpkg; then
+                echo "Package $whitelist_package already in dpkg, skipping."
+            else
+                echo "Adding to dpkg: $whitelist_package"  # Log for debugging
+                echo "$whitelist_package" >> ${TMPDIR}/dpkg
+            fi
+        done < $CONF/whitelist
+
+        # Log the contents of dpkg after appending the whitelist
+        echo "Added whitelist:"  # Log for debugging
+    else
+        echo "Whitelist not found."  # Log for debugging
+    fi
+
+    # Ensure dpkg is sorted again after whitelist is added
+    sort ${TMPDIR}/dpkg -o ${TMPDIR}/dpkg
 }
 
 # Create a blacklist of single package names from regexps in original blacklist
 # any sets such as kde/ are converted to single package names in the process
 # the final list will be used by 'applyblacklist' later.
 function mkregex_blacklist() {
-	# Check that we have the files we need
-	if [ ! -f ${WORKDIR}/pkglist ] || [ ! -f ${CONF}/blacklist ];then
-		return 1
-	fi
-
-	# Create tmp blacklist in a more usable format
-	sed -E "s,(^[[:blank:]]+|[[:blank:]]+$),,
-		/(^#|^$)/d
-		s,^, ,
-		s,$, ,
-		s,^ (extra|pasture|patches|slackware(|64)|testing)/ $,^\1 ,
-		s,^ ([^/]+)/ $, \\\./$PKGMAIN/\1\$,
-		" ${CONF}/blacklist > ${TMPDIR}/blacklist.tmp
-
-	# Filter server and local package lists through blacklist
-	( cat ${WORKDIR}/pkglist
-		printf "%s\n" $ROOT/var/log/packages/* |
-			awk -f /usr/libexec/slackpkg/pkglist.awk
-	) | cut -d\  -f1-7 | grep -E -f ${TMPDIR}/blacklist.tmp |
-		awk '{print $2}' | sort -u | sed "s,[+],[+],g
-		s,$,-[^-]+-($ARCH|noarch|fw)-[^-]+,g" > ${TMPDIR}/blacklist
+    # Check that we have the necessary files
+    if [ ! -f ${WORKDIR}/pkglist ] || [ ! -f ${CONF}/blacklist ]; then
+        return 1
+    fi
+
+    # Create a temporary blacklist in a more usable format
+    sed -E "s,(^[[:blank:]]+|[[:blank:]]+$),,
+        /(^#|^$)/d
+        s,^, ,
+        s,$, ,
+        s,^ (extra|pasture|patches|slackware(|64)|testing)/ $,^\1 ,
+        s,^ ([^/]+)/ $, \\\./$PKGMAIN/\1\$,
+        " ${CONF}/blacklist > ${TMPDIR}/blacklist.tmp
+
+    # Filter server and local package lists through blacklist
+    ( cat ${WORKDIR}/pkglist
+      printf "%s\n" $ROOT/var/log/packages/* |
+        awk -f /usr/libexec/slackpkg/pkglist.awk
+    ) | cut -d\  -f1-7 | grep -E -f ${TMPDIR}/blacklist.tmp |
+        awk '{print $2}' | sort -u | sed "s,[+],[+],g
+        s,$,-[^-]+-($ARCH|noarch|fw)-[^-]+,g" > ${TMPDIR}/blacklist
+
+    # Now exclude whitelist packages from being blacklisted
+    if [ -f ${CONF}/whitelist ]; then
+        while read whitelist_package; do
+            # If the package is in the blacklist, remove it
+            sed -i "/^$whitelist_package-[^a-zA-Z0-9 -]\+/d" ${TMPDIR}/blacklist
+        done < ${CONF}/whitelist
+    fi
 }
 
 # Blacklist filter
diff -Naupr slackpkg/files/slackpkg.conf.new slackpkg-new/files/slackpkg.conf.new
--- slackpkg/files/slackpkg.conf.new	2021-05-31 23:24:32.073866268 +0300
+++ slackpkg-new/files/slackpkg.conf.new	2025-04-03 12:01:16.743026511 +0300
@@ -1,7 +1,7 @@
 #
 # /etc/slackpkg/slackpkg.conf
 # Configuration for SlackPkg
-# v15.0
+# v1234.0
 #
 
 # SlackPkg - An Automated packaging tool for Slackware Linux
diff -Naupr slackpkg/files/whitelist.new slackpkg-new/files/whitelist.new
--- slackpkg/files/whitelist.new	1970-01-01 02:00:00.000000000 +0200
+++ slackpkg-new/files/whitelist.new	2025-04-03 11:37:55.998264648 +0300
@@ -0,0 +1,3 @@
+# Add here packages that you DONT want to be blacklisted even if all set include them is in blacklist.
+# Assume in /etc/slackpkg/blacklist you have: kde/
+# Add here any package name from kde set you DONT what to be blacklisted:
diff -Naupr slackpkg/slackpkg.SlackBuild slackpkg-new/slackpkg.SlackBuild
--- slackpkg/slackpkg.SlackBuild	2024-07-20 20:09:32.800185690 +0300
+++ slackpkg-new/slackpkg.SlackBuild	2025-04-03 12:01:40.664027241 +0300
@@ -20,12 +20,16 @@
 #  OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF
 #  ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 
+# == April 2025 == #
+# == These version is UNOFFICIAL and include modifications by Ioannis Anagnostakis == #
+# == Supported /etc/slackpkg/whitelist, no issues with slackpkgpluss == #
+
 cd $(dirname $0) ; CWD=$(pwd)
 
 PKGNAM=slackpkg
-VERSION=${VERSION:-15.0.10}
+VERSION=${VERSION:-1234.0}
 ARCH="noarch"
-BUILD=${BUILD:-4}
+BUILD=${BUILD:-1}
 
 # If the variable PRINT_PACKAGE_NAME is set, then this script will report what
 # the name of the created package would be, and then exit. This information
@@ -55,7 +59,7 @@ chown root:root $PKG/usr/sbin/slackpkg
 
 # Prepare /etc directory:
 mkdir -pv $PKG/etc/slackpkg
-cp -av blacklist.new slackpkg.conf.new post-functions.conf-sample \
+cp -av blacklist.new whitelist.new slackpkg.conf.new post-functions.conf-sample \
   $PKG/etc/slackpkg
 chmod 644 $PKG/etc/slackpkg/*
 chown root:root $PKG/etc/slackpkg/*

```
---

## Install

clone repo, run SlackBuild, upgradepkg and run slackpkg new-config;slackpkg update 

---

### trademarks

Slackware is a [trademark](http://www.slackware.com/trademark/trademark.php) of Patrick Volkerding. <br>
©[Slackpkg](https://slackpkg.org/) Some Rights Reserved - Github [page](https://github.com/rworkman/slackpkg)

