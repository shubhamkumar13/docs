# TL;DR
- cell's make set get can be replaced by atomic's make set get
- unbounded channels can be helpful in replacing worker queues
- setting up a task pool can be very helpful since spawning and joining should be controlled by main thread and not the detached function since it's a costly process.
  Previously `detach` function kept a track of spawning threads and closing them, internally.
- since we have a task library in domainslib now making a promise can be easily done

# The following points explain the tl;dr above :

- Initially I didn't realize the `CELL` struct present in the `lwt_preemptive` has a similar structure to `atomic` in multicore standard library. I tried replacing `CELL` and the functions present in the `lwt_preemptive.ml` file with Channel's `recv` and `send`.
  I had planned to try atomics, didn't try it because wasn't comfortable with the atomics api.

- Bounded channels were a problem since the task queues can be of arbitrary sizes. I tried solving the problem by creating a queue of `Mvar` like channels (channels with size 1). Domains seem to block the channels for a considerable amount of time if number of tasks didn't correspond to the number of channels. I think my code was unable to send tasks to the empty channels, I was unable to debug this.
  The `task_pool` branch now has a way to make unbounded and bounded channels, which might be helpful to replace the worker queue present in the current code.
  
 - Spawning and joining seemed like a problem but the new task_pool api can solve that problem.
 
 - Creating an async-await library helps with promises which I was previosuly trying to just use `Lwt.wait()` to receive the value in the resolver, using channel's `recv` function. But domainslib now has a dedicated library for it, which maks it easier.

- I'm still not sure how atomics work, although I would like to experiment it to replace things done with `CELL` struct. I'm hesitant to do this because I'm pretty sure I don't understand atomics. But on the surface they have seem to do the same thing so I'm more inclined to try it.
