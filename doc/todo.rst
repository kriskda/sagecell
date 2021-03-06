The next phase
==============

Scaling
-------
* (Drew) rate limiting for incoming computations and permalink requests, both total and by IP

  * HAProxy (upgrade to 1.5dev): http://blog.serverfault.com/2010/08/26/1016491873/ or http://blog.exceliance.fr/2012/02/27/use-a-load-balancer-as-a-first-row-of-defense-against-ddos/ or https://code.google.com/p/haproxy-docs/wiki/rate_limit_sessions
  * iptables: http://www.debian-administration.org/articles/187 or http://penguinsecurity.net/wiki/index.php?title=The_iptables_Rate-Limiting_Module (for example)
* load testing (ab, httperf, jmeter, our own multimechanize solution)
* Set up nginx to serve static assets


Security
--------
* (Alex K.) Implement untrusted account restrictions:

  * explore SELinux or some other solution for untrusted users (like LXC):
    http://docs.fedoraproject.org/en-US/Fedora/13/html/Security-Enhanced_Linux/sect-Security-Enhanced_Linux-Targeted_Policy-Confined_and_Unconfined_Users.html,
    http://selinux-mac.blogspot.com/2009/06/selinux-lockdown-part-four-customized.html,
    http://www.gentoo.org/proj/en/hardened/selinux/selinux-handbook.xml, http://debian-handbook.info/browse/wheezy/sect.selinux.html
  * see also Linux containers: http://www.docker.io/, http://pyvideo.org/video/1852/the-future-of-linux-containers.  see also http://www.ibm.com/developerworks/linux/library/l-lxc-security/
  * have a pool of user accounts to execute code in.  Have the forking kernel manager drop privileges when forking and switch users to an unused user account, then clean up any files by the user when the computation is done. (see https://github.com/sagemath/sagecell/issues/233)
  * Here's another crazy idea for managing diskspace: per-process filesystem namespaces (like http://glandium.org/blog/?p=217) along with copy-on-write unionfs mounts.  When a sage process is done, just throw away the unionfs mount, and poof, everything is gone.  See also http://www.ibm.com/developerworks/linux/library/l-mount-namespaces/index.html or http://blog.endpoint.com/2012/01/linux-unshare-m-for-per-process-private.html and the bottom of http://linux.die.net/man/2/mount
* Manage daemons without screen (using https://pypi.python.org/pypi/python-daemon/, maybe?  Or maybe http://www.jejik.com/articles/2007/02/a_simple_unix_linux_daemon_in_python/ ?  See also http://www.python.org/dev/peps/pep-3143/ ). (see https://github.com/sagemath/sagecell/issues/253 and https://github.com/sagemath/sagecell/issues/120)

Codebase
--------
* (Joel) implement cookie-based TOS agreement, activated on evaluating your first computation (necessary for hosting at UW)
* Change output model so that output of a request can be confined, and
  the browser knows where the output goes (instead of just trusting
  the python side to send an output id).  This would help with displaying errors, for example. (see https://github.com/sagemath/sagecell/issues/387)
* Automatically expire and restart idle workers.  See https://github.com/sagemath/sagecell/issues/391

Permalink database
------------------
* Either implement database mirroring or use a server that does, deploy multiple permalink servers.  Possibilities for a database server include:

  * web side in Tornado, Go, or Node.js (for many simultaneous connections, but I suppose we could go back to flask/wsgi or something of that nature too)
  * database side in PostgreSQL (with propogation), Redis, Couchbase, Cassandra (Cassandra seems to fit the distributed, no-single-point-of-failure need)
  * Another possibility is to do the permalink server as a simple Google App Engine project.  William has lots of credit for this sort of thing.
* logging of all requests (separate from permalinks): 
  * python logging facility (load-test this)
  * straight to append-only file
  * to database?

Lower priority
==============

Interacts
---------
* Port over William's interact implementation
* Implement William's exercise decorator
* look into putting output in iframe to avoid all of the styling
  issues we've dealt with
* Set up dependency management on python side: control updates are sent immediately on variable assignment.  This also allows us to easily track which controls got updated so they update only once, if wanted
* explore javascript widgets
* explore using bootstrap to lay out widgets (see William's design)
* implement an html layout

Slider('x+y') -> 

@interact
def update(slider):
     pass
register an update f.slider = x+y when x is changed
register an update f.slider = x+y when y is changed

@interact
class interact:
    update function

    set function


vs. my way:

Slider('x+y') -> 

the widget knows how to parse string expressions, and registers itself for updates.  It can also have a setting method

With my way, updating a slider calls just the slider update function.  Williams way, updating a slider calls the whole control group update function

With my way, the grouped variables are stored in an interactive namespace.  William's they are implicitly stored as locals of a function.   William's way is more implicit.

Deployment
----------

* Use virt-install and a kickstart file to configure centos from
  scratch to a running system:
  http://www.cyberciti.biz/faq/kvm-virt-install-install-freebsd-centos-guest/
  http://www.cyberciti.biz/faq/kvm-install-centos-redhat-using-kickstart-ks-cfg/
* Look into ansible to configure an image


Done
====
* Set up multiple servers talking to the same database (possibly distributed) over the web
* (Henry) permalinks only requested when wanted (hide div, requested and shown when you click on permalink) (see https://github.com/sagemath/sagecell/issues/350)
* (Ira) pressing evaluate multiple times really fast hangs things.  When I press evaluate a second time, before a reply message comes back, something seems to be getting messed up. (See https://github.com/sagemath/sagecell/issues/389)


Library of exercises
====================

%exercise
title    = r"Find a vector"
rank = randint(2,4)
A        = random_matrix(QQ,5,algorithm='echelonizable', rank=rank,upper_bound=10)
kernel = A.T.kernel()
question = "Find a basis for the nullspace of $%s$.  Your answer should be a list of vectors (e.g., '[(1,2,3), (3,2,1)]' )"%latex(A)
def check(a):
    try:
        a = sage_eval(a)
    except:
        return False, "There was an error parsing your answer. Your answer should be a list of vectors (e.g., '[(1,2,3), (3,2,1)]' )."
    i = [vector(QQ,j) for j in a]
    v = span(i)
    if v.dimension()!=len(i):
        return False, "Are your vectors linearly independent?"
    elif v != kernel:
        return False, "You are missing some vectors"
    else:
        return True, "Great job!"
hints = ["The RREF is $%s$."%latex(A.rref())]
hints.append(" ".join(hints)+"  The nullity is %d."%kernel.dimension())
