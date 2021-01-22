httpd-config

The Apache (httpd) server config files reworked.

==== Contents:

-  Why another Apache Configuration
-  What is good, and not
-  Using this setup
-  Virtual host duality
-  Configuration file map
-  The Hydra
-  Helper scripts
-  Names and paths for vhosts
-  Drop-in directories
-  Other config directories
-  The `DocumentRoot`
-  The `cgi-bin` directory

====  Why another Apache Configuration

After using a variety of configuration files, from several different Linux
distros and seeing their concepts in play, I've developed a few preferences.
I've seen some ideas I liked and others I didn't like so well. The major players
in my exposure have been openSUSE, Ubuntu, CentOS and Arch Linux. I give credit
to all of them for having some good ideas, even if they are not ones I've used
here.

==== What is good, and not

Some of the preferences and ideas I dealt with are:

    *  I don't like monolithic files.
    *  I do like well documented files.
    *  I like files to be named with meaningful names
    *  I like files that are single-purpose
    *  I like files which are interchangeable
    *  In a system of interconnected files and settings I like having a single
       point of control
    *  I like plugins and drop-ins
    *  I don't like surprises
    *  I like to use newer methods
    *  I like fallbacks and failsafes
    *  I like standards
    *  I don't like something to look standard while not being true to the
       standard
    *  I like to be obvious about not matching the standard when I don't
    *  I like to avoid problems from other sources, and try to plan for them
    *  I don't like foreign packages making unadvertised changes to files I've
       built
    *  I like having backups, lots of backups

The last one is the main reason for this repository. I don't expect that anyone
else will find my setup enticing, or think of it as the best way to do it. I
    like backups, and having this on GitHub gives me a good backup. Nothing
    I've done is secret, nor especially unique. I've just done some cherry-
    picking of other peoples' ideas and created my own patchwork quilt. In the
    spirit of the story behind the name of Apache, patching things together
    seems most appropos.

==== Using this setup

Using this configuration setup is rather simple, if requirements are met.

    1.  Knowing what the directives mean, and how to use them, is important
    2.  The config directory which was installed with Apache gets replaced
    3.  This probably will _not_ work if it's just dropped into the /etc
        directory
    4.  As written, the server will serve nothing, unlike the norm of having a
        default site when all others are undefined
    5.  The DocumentRoot is _not_ the standard place (there isn't one), nor is
        it in a common or typical place
    6.  Knowing where updates will go, and watching what they change will be
        part of the maintenance of the server.
    7.  Except for Arch Linux (and possibly spin-offs from there) where the
        config files are typically won't be written over during upgrades.
    8.  Other packages which make changes to the directory for additonal
        controls _should_ be safe and not overwrite these settings

==== Virtual host duality

One of the concepts I liked was a pair of directories for the virtual hosts.
Ubuntu uses sites_available to hold the *.conf file for every site the server
might serve. When the admin wants to make that site active a symlink to the
config file is made in the sites_enabled directory. The benefit is that the
config file always exists, even when it's not active.

==== Configuration file map

Along the same lines, Ubuntu uses the same process for config files relative to
each module, or module grouping, with the config file being in an 'available'
directory and symlinked on need into an 'enabled' directory. I don't like
treating the modules in that fashion. Instead I prefer to have the module
loading and configurations react to settings in a common file to control
options, and reconfigured on a per-vhost basis where appropriate. The system
for doing this is well embodied in the openSUSE setup. Another thing I liked
    from the openSUSE setup is having the loading order and filesystem layout
    described in their main config file, at the top. That's been replicated
    here as well.

==== The Hydra

The part I don't like about openSUSE's implementation is the possibility of
configuration settings being read from two possible locations with conflicting
directives and no simple way for the admin to be sure which one is being used.
That option, while nicely integrated into the openSUSE settings menus, has often
confused my reading of what _should_ be running. Limiting the choice of modules
to load, and other settings to one set of files works best for me. The
config-options.conf and httpd-modules.conf files fit that bill. The
config-options.conf file is intended to control which optional modules are
loaded, or loadable by vhosts, and the modules file is inteded to load the ones
which are needed, and select the optional ones based on settings in the
config-options.conf file. I'm sure that there are more options available, or
finer grained control available, I just haven't gotten that deep into it. The
vhost files can load, or not, different settings, but their availability should
be under control of the options file. For example, a vhost file could try to
load the WebDAV settings, but the controlling file still checks the DAV_SERVER
option before actually setting the directives for it.

==== Helper scripts

Unlike the Ubuntu system's full-featured script, I've made a trio of simple
scripts to enable, and disable, vhosts and to create them as needed. In all
three cases the one and only argument is the FQDN of the vhost, and it's only
built to handle three levels of naming: 'host.domain.tld'. A name without a host
will break the scripts and fourth level host name will "work" in the scripts,
but break the organization of the system. Modification of the trio to handle a
fourth level, or higher, host name is possible, should the need arise.

==== Names and paths for vhosts

In dealing with multiple registered domains with multiple host names therein,
I've developed the habit of using reversed order on the names. 'host.domain.tld'
uses a vhost file of 'tld.domain.host.vhost.conf', or
'tld.domain.host.vhost.ssl.conf' for an SSL vhost. And the files served from the
'tld/domain/host/htdocs' directory. Doing things that way allows me to remove an
entire domain, or move it to another server, without having to dig through the
files to find what I need. Everything naturally sorts in directory listings to
match the layout. It's good enough for the in-addr.arpa. organization it should
be good enough to handle anything I could make.

==== Drop-in directories

Another point above is that I don't like it when adding or updating a package,
the main server itself or some addon, plugin, or tool, makes unadvertised
changes to files I've taken the time to create. Especially disturbing is when
those changes end up being breaking changes. The Apache ecosystem seems to have
half-way settled that problem by allowing independent packages to 'drop' config-
uration files into a pre-determined directory which the main config file is
supposed to read. Unfortunately, in my experience, there are two choices, not
one. Some packages expect their files to land in the 'extra' directory and
others want to use the 'conf.d' directory. I personally think the 'extra' option
is an older one which survives due to the amount of "old" servers in the wild,
and that 'conf.d' is the newer variation, which is becomming more entrenched
with many services and systems which need distributed config files and want to
avoid the effect I don't like.

I've seen the '.d' suffix on directories serving two different purposes. One use
is as a 'drop-in' place for extenal packages to add their own config settings to
a package to be automatically loaded by the main package. The same automatic
inclusion is also applied to files the system admin might create. A good example
of both cases is the settings for Grub in the /etc/grub.d/ directory. The other
purpose is a location to place config files which may, or may not, be loaded by
the application on a selctive basis. Prime example of this use case is the
/etc/profile.d/ directory where which files are loaded is determined by the
profile script.

To avoid complications, and allow the greatest freedom for adding packages
later, I've elected to include both /etc/httpd/extra/ and /etc/httpd/conf.d/
in the processing of the main config file. I've also choosen to use conf.d as
the directory for the config snippets to include in the main config file while
still keeping them under the control of the selected options.

==== Other config directories

Taking a clue from ArchLinux I moved the entire main configuration one level
deeper, into the /etc/httpd/conf/ directory. For safety sake there is an extra
directory in the level-down conf directory as well, but it should remain empty.
So, the main config file is /etc/httpd/conf/httpd.conf and the programaticlly
included config files are /etc/httpd/conf/conf.d/*.conf.

The vhost config files are in /etc/httpd/sites_available/` with the activating
symlinks in /etc/httpd/sites_enabled. With many packages possibly expecting
vhosts to be in /etc/httpd/vhosts/, I've symlinked that to the activating
directory as well. Mostly for convenience I place the icons, manual, and default
error page directories in the server root, /etc/httpd/. The hope is that as the
server recieves updates, those will recieve updates, and that if they don't they
will be easy to remember locations for manually doing the updates.  Lastly, I
added a symlink, for admin convenience, to the root of the structure holding the
DocumentRoot directories, /content/sites/ as /etc/httpd/sites-root/.

==== The `DocumentRoot`

As to the location of the served files I was split on several options. I've seen
them as a directory in /srv/, or /srv/www/. I've also seen them as
/etc/apache2/htdocs/, and read several other suggestions as to where they
'belong,' none of which agree, even in principle, except on the point that they
don't fit anywhere in the "standard" for Linux file system layouts. That's not
entirely true, of course. Version 2.3 of FHS does say that /srv/ is for "Data
for services provided by this system" and suggests that web servers fit in that
    category. It does go on, however, to state that where, within /srv/, to put
    what remains undecided. "The methodology used to name subdirectories of
    /srv is unspecified as there is currently no consensus on how this should be
    done." One of my points at the beginning is that I don't like things to
    "look" like they are standard when in truth they are not. Even though /srv/,
    with an undefined structure, is included in FHS v2.3, it still feels like a
    non-standard so far. For that reason I've decided to use an obviously non-
    standard directory, in an obvious place. If, at some future date, FHS
    cements the standard for webserver documents, a single change in the
    `config-variables.conf` file followed by moving the `/content/sites/` tree
    to the new location will make this setup comply with that FHS decision.

Placing the served documents anywhere within the /etc/ tree seems like a very
bad idea. The server security is not guaranteed to be perfect, and any leaking
of data from the /etc/ tree could be bad. Using the /srv/ tree seems reasonable;
they are 'server' files. The data files for MySQL often go there, as does the
FTP public root. I'm not sure what else might be placed there by other packages;
perhaps even git server uses that location. In any case, I prefer to have things
segmented in a meaningful way, and using /srv/ does not seem to fit that
description. On many shared hosting environments I have seen /home/, or even
/home2/ used as the root; using various schemes to organize the "users" of the
hosting service. In a shared hosting environment that even makes some sense;
each account is a "user" of the system and is serving their own files from a
common server. That's not so good outside of a shared hosting envrionment,
however. Double that if the server is also used as a desktop for a user, or
users. Again, that risks a leak similar to that of using /etc/. Though, in this
case it would be personal information leaked rather than system data. Still, not
a good thing.

In the end, accepting that there is no place within the FHS specifically called
out for web server data, and it has to be placed somewhere, I went with some-
thing I've encountered in one shared hosting environment. With a small change.
That host used /content/ as the absolute root, created subdirectories based on
user name. Using my username here, the root for my hosted site would have been
/content/c/h/i/chindraba-work/htdocs/. My modified version of the config is
aimed at using one server with several vhosts, without any reference to "users"
or other classifications. Adopting the naming of the vhost config files by
reversing the FQDN naturally fits into using the same patter for naming direc-
tories, just placing the root of the tree in /content/sites/. Thus, the domain
host.sample.org would have a DocumentRoot of
/content/sites/org/sample/host/htdocs/ and a cgi-bin aliased to
/content/sites/org/sample/host/cgi-bin/.

==== The `cgi-bin` Directory

The last directory brings up another point. It seems common, though not univer-
sal, to have the cgi-bin for a virtual host within that vhost's DocumentRoot, as
./htdocs/cgi-bin/ next to ./htdocs/index.html. My non-professional opinion is
that having the scripts "outside" the, often insecure, DocumentRoot is an avoid-
able risk. Many packages, such as e-commerce and forums, use PHP scripts and
have files with credentials for accessing the database, if nothing else, some-
where within those files. If such files are within the DocumentRoot of the
server they might be "served" by Apache. Moving them up one directory in the
tree places them back within the restrictive settings applied to the rest of the
filesystem. A separate Directory directive can assign the proper permissions to
that directory and not fall under the .htaccess file in the DocumentRoot;
commonly written, and broken, by inexperienced "admins" and hobbyists.
