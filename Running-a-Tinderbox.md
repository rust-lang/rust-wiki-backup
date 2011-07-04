Running a tinderbox is very simple:

* Get a rust build environment working first. Like, get your own system through 'make check'
* `hg clone http://hg.mozilla.org/users/graydon_mozilla.com/rust-tinderbox`
* `cd rust-tinderbox`
* `vi rust-tinderbox.py`:
    * edit `smtpserver` to some working SMTP relay you can send mail through
    * edit `username` to your email address (if you want to; it controls who you're sending email 'From')
* `./rust-tinderbox.py`
 
From time to time you might also want to run `hg pull -u` to get fresher tinderbox code as we adjust it; it changes slowly but it will happen from time to time. Join IRC to ask if the scripts stop working.