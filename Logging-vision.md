# Logging

## The current situation

(as per May 30 2011)

* There are two logging primitives `log` and `log_err`. Both allow a variety of types to be given.

* `log_err` output is always printed. This is typically used right before a `fail`, or in case of other serious problems.

* `log` is a conditional logging statement. Its output is suppressed by default, but can be turned on using the `RUST_LOG` environment variable.

* `RUST_LOG` takes a comma-separated list of directives. A directive is, for now, a module name prefix, for example `std::io`. The 'main' program will live under `main`, unless an explicit crate name is given in the crate file.

* In the runtime, the `LOG` and `LOG_ERR`, (or `DLOG_[ERR]` if no task pointer is available) serve a similar role as `log` and `log_err`. They explicitly set their own 'module'. These modules live under `rt`, for example `rt::comm`, `rt::upcall`.

* Logging (when muffled) is really really cheap in rustc now.

## The future

* Logging will be done through a `#fmt`-like syntax extension, which (somehow) expands to a special form that handles the module-check wiring and such. The syntax extension will handle stringification, so the runtime logger will only deal with strings anymore.

* There will be more log levels: 0 error, 1 warning, 2 note, 3 debug. The default log level will be 1 (show warnings and errors). The `RUST_LOG` directives will support `module=2` syntax to set specific log levels. If no `=X` is given, the level for that module is set to 3.

* There will be an API (though a library call) to restrict logging to a single domain (or maybe task). (Maybe also a set of domains, if those become referenceable somehow.)

* There will also be an API that allows you to pass in your on `RUST_LOG`-style string to reset the per-module logging levels at runtime.
