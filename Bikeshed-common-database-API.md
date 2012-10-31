We should provide a common interface for databases to ease porting from one database backend to another.

## Similar projects for other languages:

* Python:
  * [DB-API](http://www.python.org/dev/peps/pep-0249/)
* Go:
  * [sql](http://golang.org/pkg/database/sql/)
* Ruby:
  * [DataObjects](https://github.com/datamapper/do)
  * [Sequel](http://sequel.rubyforge.org/)
* Node.js:
  * [Sequelize.js](http://www.sequelizejs.com/)
* Java:
  * [JDBC](http://www.oracle.com/technetwork/java/overview-141217.html)
* C++:
  * [QtSql](http://doc.qt.digia.com/4.5/qtsql.html)
  * [SOCI](http://soci.sourceforge.net/)
* Objective C:
  * [Core Data](https://developer.apple.com/library/mac/#documentation/Cocoa/Conceptual/CoreData/cdProgrammingGuide.html)
  * [BaseTen](http://basetenframework.org/) (Seems to be offline)

## Known bindings:

* Sqlite:
  * https://github.com/linuxfood/rustsqlite 
* PostgreSQL (in progress):
  * https://github.com/erickt/rust-pg

Needed:

* MySQL
* Oracle

## Open questions:

* Should we bother trying to support NoSQLs?
* Do we low-level like Python's DB-API? Or higher level, like Ruby's Sequel?