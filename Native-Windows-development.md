# Environment Setup
1. Install the [[Mozilla-Build enviornment|http://ftp.mozilla.org/pub/mozilla.org/mozilla/libraries/win32/MozillaBuildSetup-Latest.exe]] (referred to as m-b)
2. Install [[Ocaml|http://caml.inria.fr/pub/distrib/ocaml-3.11/ocaml-3.11.0-win-mgw.exe]]
3. Install [[Git|http://msysgit.googlecode.com/files/Git-1.7.3.1-preview20101002.exe]]
    * In the installer, don't have git autoconvert line endings. Just use a good text editor like vim or emacs.
4. Add the Ocaml and Git paths to the PATH environment variable used by the m-b environment. m-b likes to reset the PATH when it starts you'll want to edit %USERPROFILE%/.bash_profile (~/.bash_profile if you're in the m-b environment). The m-b default shell is bash 3.1.
    * For git, add the Git/cmd directory instead of the Git/bin directory to avoid any conflicts with the existing tools in the m-b environment.
5. In m-b's /bin/ directory (or somewhere on your path), you'll want to add a script named 'git' which will invoke the CMD.exe shell script which was installed in step 3:
```python
     #!python
     import os
     import subprocess
     import sys
     
     path = "%s\Git\cmd\git.cmd" % os.environ['PROGRAMFILES']
     
     cmdline = [path]
     cmdline.extend(sys.argv[1:])
     
     git = subprocess.Popen(cmdline)
     
     sys.exit(git.wait())
```
6. You should now be able to run `git` and `ocaml` directly from the shell
7. Apply my as-yet-unposted patches
8. `make check`