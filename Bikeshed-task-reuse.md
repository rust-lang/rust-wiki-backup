Depending on how expensive our task spawn/destroy process is, we should consider a task reuse system. I think this could be implemented almost exclusively in libcore, with very few changes to the runtime. To do this, `spawn` basically becomes as follows (where `real_spawn` is the current version of `spawn`).

```
fn spawn(f: fn~()) -> task {

    ... see if we can reuse a task instead ...

    real_spawn() {||
        f();

        let p = port();

        ... tell controller that I can get new tasks by sending them to p ...

        let mut live = true;
        while live {
            alt p.recv() {
              terminate { live = false; }
              spawn(f) {
                f();
                ... tell controller I'm done ...
              }
            }
        }
    }
}
```

This involves creating a controller task that keeps a pool of old tasks around. The controller has the option of shutting down tasks that aren't needed, or creating new tasks when new ones are needed.

I tried doing a version of this for my parallel breadth first search code, but it did not seem to make a noticeable difference in performance. Until we have evidence that this is a good idea, I don't think we should do this, but I wanted to make sure the idea was written down somewhere.