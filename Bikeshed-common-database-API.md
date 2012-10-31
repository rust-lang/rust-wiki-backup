We should provide a common interface for databases to ease porting from one database backend to another.

## Similar projects for other languages:

* Python:
  * DB-API: http://www.python.org/dev/peps/pep-0249/ 
* Go:
  * sql: http://golang.org/pkg/database/sql/
* Ruby:
  * DataObjects: https://github.com/datamapper/do
  * Sequel: http://sequel.rubyforge.org/
* Node.js:
  * http://www.sequelizejs.com/

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