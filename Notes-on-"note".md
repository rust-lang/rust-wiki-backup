This is just an idea / proposal for discussion

# note

The way I understand it the note statement will allow to cheaply record information on the stack for later logging in stack traces (i.e. upon fail).  This is an awesome idea.  However such a facility may be useful for regular logging as well.  

# NDC/MDC Use Case

Java's log4j has two facilites for recording state that gets appended automatically to all log messages that fire in the enclosed scope: NDC, a thread-local stack of values, and MDC, a string-map of values.  Both are very useful to record relevant information for log messages that do not have explicit access to required state (e.g. session id, user login name, transaction id).  Example in pseudo Java:


    try {
        MDC.put("sessionId", sessionId);
        someComponent.run(...);
    } finally { MDC.remove(sessionId); }

    // inside someComponent
    void run(...) {
       logger.info("Infomative message");
    }

When the log statement fires, "Informative message" will be extended with the `sessionId` by log4j without `someComponent` having access to it.

# Extending note

In rust, `note` could be extendend/amended with a variant, (called `note(core::info, ...)` here), that will automatically record state for appending to all log messages inside the enclosed scope (i.e. contained stack frames).

    fn outer() {
       note(core::info, #fmt["sessionId: %i", sessionId]);
       someComponent.run(...);
    }

    // inside somePluginComponent
    fn run(...) {
       info "Informative message";
    }

As above, "Informative message" would be ammended with "sessionId: <sessionid>" automatically.

There probably is an interesting interaction with log levels here, i.e. the need to specify what gets appended in what log level. It may even be possible to just understand stack trace logging as a special kind of log level (or enable assigning of log levels to fail, hmm, I never thought that before but it would capture patterns where one tries to record the severity of exceptions by catch-and-rethrow-as-apropriate).






   


