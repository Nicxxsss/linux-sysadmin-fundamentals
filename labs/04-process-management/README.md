# Inspecting and stopping Linux processes

This lab focused on identifying process states, checking resource usage and stopping processes without leaving them running in the background.

## Inspecting an idle process

I started a five-minute `sleep` process in the background:

```bash
sleep 300 &
lab_pid=$!
```

The `&` returned control of the terminal immediately. `$!` provided the PID of the most recent background process.

I inspected it with:

```bash
ps -p "$lab_pid" -o pid,ppid,user,stat,%cpu,%mem,etime,cmd
```

The process was in state `S`, meaning interruptible sleep. It used no measurable CPU because it was waiting rather than actively processing data.

The output also showed its parent process ID. In this case, the parent was the Bash shell running through my SSH session.

## Sending SIGTERM

I stopped the process with:

```bash
kill -TERM "$lab_pid"
```

Despite its name, `kill` sends a signal to a process. `SIGTERM` asks the process to terminate and gives it an opportunity to exit normally.

After waiting for the process, the shell returned status `143`:

```text
128 + 15 = 143
```

The value `15` is the signal number for `SIGTERM`. A second `ps` check confirmed that the process was no longer running.

I did not use `SIGKILL` because it stops a process immediately and should normally be reserved for a process that does not respond to a regular termination request.

## Simulating high CPU usage

I used `yes` to create a controlled CPU-intensive process:

```bash
timeout 20s yes > /dev/null &
```

`yes` continuously generates output. Redirecting it to `/dev/null` prevented the output from filling the terminal or a file. `timeout` provided a safety limit in case the process was not stopped manually.

The process tree contained two processes:

- `timeout`, which controlled the time limit;
- `yes`, which was its child and consumed approximately 99% of one CPU.

`yes` appeared in state `R`, meaning it was actively running.

## Finding the process without its PID

I listed running processes in descending CPU order:

```bash
ps -eo pid,ppid,user,stat,%cpu,%mem,etime,cmd --sort=-%cpu | head -n 5
```

This placed `yes` at the top of the output. This approach is useful when a system is slow and the PID of the responsible process is not already known.

I stopped the controller with `SIGTERM` and confirmed that neither the controller nor its child remained active.

The complete command output is available in [`process-management.txt`](../../evidence/04-process-management/process-management.txt).
