x20 is a world-wide distributed and replicated file system in prod for storing
and sharing files. LOAS credentials are usually necessary, so please run
prodaccess before trying to access your data from your workstation.


What is it good for?
--------------------

If you write large files to your home directory to share with other engineers,
then x20 is
 *  providing cheaper storage space than homedir filers
    (but more expensive than CNS)
 * faster for people accessing it from other offices
 * well suited for files between 10 MB and 10 GB
 * easily accessed from Goobuntu at /google/data/{ro,rw}/users/ra/randomuser/
 * easily accessed via a browser for reading world-readable files at
     https://x20web.corp.google.com/users/ra/randomuser/
   and for the www subdirectory at
     https://x20web.corp.google.com/~randomuser/


What is it NOT for?
-------------------

 * reading/writing many small files, <1 MB, e.g., Piper clients
 * writing a lot (number of writes and bandwidth); x20 is a shared resource
 * storing sensitive data
 * things that need full POSIX-compliance (we don't support inodes, hardlinks,
   and locking yet)


Where do I get more information?
--------------------------------

 * http://go/x20
 * for further help please email x20-discuss@google.com
