NOTE NEED EXECUTE PERMS ON:
-mgServer
-mgSubmit
-mgProcessor
-runtests
-peak
-timedCountdown

This grid system is written 100% in bash and utilizes #of processers*2 + 1 number of FIFOS.

Each processor has 2 FIFOS connected to the server in order to:
1. Recieve a task 
2. Return it's current status

The server itself has one FIFO to recieve commands from the user. 

In order to get the status from each processor, the server runs a peak function to see
what's inside the processor's status FIFO when a command is recieved in order to properly 
assign the task. This runs each time on every processor after recieving a command from the user.

This is to ensure that states are up to date when assigning tasks.

By utilizing the timeout command, the server is able to quickly check if a processor is done or not.

If the status pipe has something inside then it means the processor is finished with it's job, otherwise
the pipe is empty and the peak function will timeout, meaning the processor isn't done yet. This resolves 
the problem of halting the server and it can keep running other processes instead of waiting for the busy processor
to send something back

In order to do atomic opperations with pipes I chose to utilize the mkdir command for each processor.

Riccardo Murri answered Aug 11 '10 at 12:14
https://unix.stackexchange.com/questions/70/what-unix-commands-can-be-used-as-a-semaphore-lock

I was initially interested in the flock(1) but learnt that it wasn't portable.
Sometimes you'll see a rm has failed message but it is working as intended. 

------------------------
Instructions

1. run the server 
    ./mgServer
2. run the submiter with whatever command
    ./mgSubmit ls
3. get status by
    ./mgSubmit -s
3. exit by 
    ./mgSubmit -x

------------------------
Testing

Utilizes the default given timedCountdown program for testing.

Note: occasional variance with actual results but runs similar most of the time

My first test involves running the timedCountdown program 11 times getting the status.
This will test if the program is able to properly assign and queue jobs when issued.

The test passes if you see that the status shows 8 completed jobs when status is asked. 
The remainder 3 tasks finishes executing shortly after.

Result: succeeded

Expected result:
Sending './timedCountdown' to Processor 1
Processor 1 executing './timedCountdown'...
Sending './timedCountdown' to Processor 2
Processor 2 executing './timedCountdown'...
Sending './timedCountdown' to Processor 3
Processor 3 executing './timedCountdown'...
Sending './timedCountdown' to Processor 4
Processor 4 executing './timedCountdown'...
Sending './timedCountdown' to Processor 5
Processor 5 executing './timedCountdown'...
Sending './timedCountdown' to Processor 6
Processor 6 executing './timedCountdown'...
Sending './timedCountdown' to Processor 7
Processor 7 executing './timedCountdown'...
Sending './timedCountdown' to Processor 8
Processor 8 executing './timedCountdown'...
All processors are currently busy
Waiting...
Processor 1 done './timedCountdown'
Processor 2 done './timedCountdown'
Processor 3 done './timedCountdown'
Processor 4 done './timedCountdown'
Processor 5 done './timedCountdown'
Sending './timedCountdown' to Processor 1
Processor 1 executing './timedCountdown'...
Processor 6 done './timedCountdown'
Sending './timedCountdown' to Processor 2
Processor 2 executing './timedCountdown'...
Processor 7 done './timedCountdown'
Sending './timedCountdown' to Processor 3
Processor 3 executing './timedCountdown'...
Processor 8 done './timedCountdown'
Number of processors: 8
Number of jobs processed: 8
Processor 1: busy
Processor 2: busy
Processor 3: busy
Processor 4: idle
Processor 5: idle
Processor 6: idle
Processor 7: idle
Processor 8: idle

My second test involves running the timedCountdown program 19 times getting the status then shutting down.

This will test if the program is able to properly assign and queue jobs when all cores are exhausted and a shutdown
is issued in the middle of running.

The test passes if 27 jobs are completed by the status check and the 3 remaining finish before the system shutsdown.

Result: succeeded

Expected result:
Sending './timedCountdown' to Processor 1
Processor 1 executing './timedCountdown'...
Sending './timedCountdown' to Processor 2
Processor 2 executing './timedCountdown'...
Sending './timedCountdown' to Processor 3
Processor 3 executing './timedCountdown'...
Sending './timedCountdown' to Processor 4
Processor 4 executing './timedCountdown'...
Sending './timedCountdown' to Processor 5
Processor 5 executing './timedCountdown'...
Sending './timedCountdown' to Processor 6
Processor 6 executing './timedCountdown'...
Sending './timedCountdown' to Processor 7
Processor 7 executing './timedCountdown'...
Sending './timedCountdown' to Processor 8
Processor 8 executing './timedCountdown'...
All processors are currently busy
Waiting...
Processor 1 done './timedCountdown'
Processor 2 done './timedCountdown'
Processor 3 done './timedCountdown'
Processor 4 done './timedCountdown'
Processor 5 done './timedCountdown'
Sending './timedCountdown' to Processor 1
Processor 1 executing './timedCountdown'...
Processor 6 done './timedCountdown'
Sending './timedCountdown' to Processor 2
Processor 2 executing './timedCountdown'...
Processor 7 done './timedCountdown'
Sending './timedCountdown' to Processor 3
Processor 3 executing './timedCountdown'...
Processor 8 done './timedCountdown'
Sending './timedCountdown' to Processor 4
Processor 4 executing './timedCountdown'...
Sending './timedCountdown' to Processor 5
Processor 5 executing './timedCountdown'...
Sending './timedCountdown' to Processor 6
Processor 6 executing './timedCountdown'...
Sending './timedCountdown' to Processor 7
Processor 7 executing './timedCountdown'...
Sending './timedCountdown' to Processor 8
Processor 8 executing './timedCountdown'...
All processors are currently busy
Waiting...
Processor 1 done './timedCountdown'
Processor 2 done './timedCountdown'
Processor 3 done './timedCountdown'
Processor 4 done './timedCountdown'
Processor 5 done './timedCountdown'
Sending './timedCountdown' to Processor 1
Processor 1 executing './timedCountdown'...
Processor 6 done './timedCountdown'
Sending './timedCountdown' to Processor 2
Processor 2 executing './timedCountdown'...
Processor 7 done './timedCountdown'
Sending './timedCountdown' to Processor 3
Processor 3 executing './timedCountdown'...
Processor 8 done './timedCountdown'
Number of processors: 8
Number of jobs processed: 27
Processor 1: busy
Processor 2: busy
Processor 3: busy
Processor 4: idle
Processor 5: idle
Processor 6: idle
Processor 7: idle
Processor 8: idle
Shutdown command detected
Waiting for processor 1 to finish...
Waiting for processor 2 to finish...
Waiting for processor 3 to finish...
Processor 4 has stopped
Processor 5 has stopped
Processor 6 has stopped
Processor 7 has stopped
Processor 8 has stopped
Finishing up work...
Processor 1 done './timedCountdown'
Processor 2 done './timedCountdown'
Processor 3 done './timedCountdown'
System shutdown complete
