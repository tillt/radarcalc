# Sniping Radar Task Scheduler

Last War, the game - a radar task optimizing scheduler process. Right away I need to apologize for it being so complicated -- this entire thing begs to get implemented into some neat frontend. The goal here is to start working on radar missions at around the time your total amount of tasks on the radar becomes highest possible. We want that moment, where we maximize our harvest to fit our schedule and this is how to do it.

### The Data

The radar refresh period is 6 hours, that is a fixed parameter. The rest is variable and thus this process needs your input.

https://github.com/tillt/radarcalc/blob/main/site/RadarTaskCount.png
1. Check your current radar period start — when does the radar get new tasks? [Rp]
   ![Radar period starts in](https://github.com/tillt/radarcalc/blob/main/site/TimeLeftUntilStart.png)
2. Check the amount in the queue — how many radar tasks do you currently have in your queue? [q[t1]]
   ![Radar tasks queued](https://github.com/tillt/radarcalc/blob/main/site/RadarTaskQueueCount.png)
3. Check the amount on the screen — how many radar tasks do you currently have visible? [v[t1]]
   ![Radar tasks visible](https://github.com/tillt/radarcalc/blob/main/site/RadarTaskVisibleCount.png)
4. Check your radar level and its current parameters — what is the number of tasks added per refresh? [N]
   ![Radar tasks per refresh](https://github.com/tillt/radarcalc/blob/main/site/TasksPerRefresh.png)
5. The amount of radar tasks you can display at a time appears to be identical with the amount of tasks you get per refresh [V=N]? [V]
6. Check your radar level and its current parameters — what is the maximum queue size? [Q]
   ![Radar queue size](https://github.com/tillt/radarcalc/blob/main/site/RadarQueueMax.png)

The total amount of radar tasks you can accumulate before you waste (stop accumulating) is the number of tasks it can display at a time plus the number of tasks it can store in the queue. [V+Q].

6. Which of the following radar refresh windows would you like to chose for starting your Last War day with?
    * Add time left until next refresh cycle to the current time of the day.
    * Now you have the period start — add multiples of 6 hours to find your window.

### Example

* time left before radar refresh is 12minutes
* current time of the day is 15:21
* your refresh window starts at 15:33
* the schedule for the next 24hours is 15:33, 21:33, 03:33, 09:33, 15:33

## Optimizing for VS and Arms Race

Synchronize your VS radar tasks to the arms race. Many tasks need stamina, doing them within the drone-phase in the arms race is advisable.

Assume the next day’s arms race drone-phase was starting at 12:00 (noon). You would want to start popping radar tasks not before that time. That means your favorite window for maximum radar tasks in the queue is the one between 9:33 and 15:33. At that time, your would want everything to be filled to the maximum. It does also mean that at 9:33 you should get the last N tasks to fill everything up. [tx]

The rest is math - this is what this all boils down to, if I am not mistaken (there is a good chance I am!).

In this example; tomorrow morning in the window starting 9:33 [tx] we want V+Q tasks to be available to us.

* tomorrow 9:33 -> c[tx] = (V + Q) - N
* tomorrow 3:33 -> c[tx-1] = (V + Q) - 2*N
* today 21:33   -> c[tx-2] = (V + Q) - 3*N
* today 15:33   -> c[tx-3] = (V + Q) - 4*N
* => x = 4

Now plugging in some real data to play it all through.

* Q = 40 (queue size)
* N = 11 (added per cycle)
* V = 11 (visible at a time)
* q[t0] = 10 (currently queued)
* v[t0] = 11 (currently visible)
* => **c[t0] = q[t0] + v[t0]** = 21 (currently available)
* => V+Q = 51 (maximum amount possibly available)
* => V+Q = c[t0] + x*N (the maximum amount should be happening in x phases)

Now we can calculate the optimal setup for now, lets call it c[t0]'. c[t0] is what we see at the moment in contrast to c[t0]' which is what we want to see to make things fit perfectly well.

* => **c[t0]' = (V + Q) - x*N** (we get x from our chosen window above)
* => c[t0]' = (40 + 11) - 4 * 11
* => c[t0]' = 7

Lets validate that...

* now 15:21 -> c[t0] = 21
* today 15:33 -> c[t1] = 32
* today 21:33 -> c[t2] = 43
* tomorrow 03:33 -> c[t3] = 54
* tomorrow 09:33 -> c[t4] = 65
* => I am 14 tasks over what I want them to be - I would waste tasks if I did not solve some before 9:33 tomorrow morning.
* => Ideally, right now I work through all 11 visible tasks and get number of available tasks down to 7 [c[t0] = c[t0]' = 7].
