# micropython-aioschedule

Overview
-----

Run Python coroutines periodically using a friendly syntax.

- Asynchronous using asyncio
- A simple to use API for scheduling jobs
- In-process scheduler for periodic jobs
- Persistence, tasks survive reboot/restart/crash
- Tested on MicroPython 1.19

Based on the python schedule library by dbader https://github.com/dbader/schedule

Background
-----

This project is very much a work in progress, having been ported from CPython it is not very "micro" and the codebase needs to be reduced. All issues/pr's welcome.

The goal of this library is to create a simple micropython scheduler that is asynchronous and persistent (tasks are stored in flash and can be reloaded after a reboot/restart/crash). This is important for devices that operate on battery power and need to conserve power. Devices can be put to deep sleep when no tasks are scheduled to run.

At this point in the time the library is designed to call async coroutines, in future non-async calls may be supported.


Requirements
-----

Requires datetime and functools from micropython-lib

Automatic installation, requires MicroPython from github (later than 30-09-2022)
```python
>>> import mip
>>> mip.install("github:ThinkTransit/micropython-aioschedule")

Installing github:ThinkTransit/micropython-aioschedule/package.json to /lib
Copying: /lib/schedule/schedule.py
Installing functools (latest) from https://micropython.org/pi/v2 to /lib
Copying: /lib/functools.mpy
Installing datetime (latest) from https://micropython.org/pi/v2 to /lib
Copying: /lib/datetime.mpy
Done
>>> 
 
```
Manual installation
```
Copy schedule.py to your micropython lib directory
Copy functools.py and datetime.py to your lib directory from micropython-lib
```

Usage
-----

Create some tasks.

```python
import schedule

async def job():
    print("I'm working...")
schedule.every(1).minutes.do(job)

async def job_with_argument(name):
    print(f"I am {name}")
schedule.every(10).seconds.do(job_with_argument, name="MicroPython")

```

If you have an existing async application, add the scheduler task to your main event loop.
```python
import schedule
import uasyncio as asyncio

asyncio.create_task(schedule.run_forever())
```

Full simplified working example code.
```python
import schedule
import uasyncio as asyncio


async def job():
    print("I'm working...")
schedule.every(1).minutes.do(job)


async def job_with_argument(name):
    print(f"I am {name}")
schedule.every(10).seconds.do(job_with_argument, name="MicroPython")


async def main_loop():
    # Create the scheduling task
    t = asyncio.create_task(schedule.run_forever())
    await t

try:
    asyncio.run(main_loop())
except KeyboardInterrupt:
    print('Interrupted')
except Exception as e:
    print("caught")
    print(e)
finally:
    asyncio.new_event_loop()

```
Persistence
-----

To persist the schedule it must be saved to flash by calling save_flash(). Typically this should be done before your microcontroller goes to sleep so that the last run times are saved.

```python
import schedule

# Save schedule to flash
schedule.save_flash()
```

To load from flash call load_flash(). Typically this should be done when your program first boots.
```python
import schedule

# Load schedule from flash
schedule.load_flash()
```

If your device is permanently running it is still a good idea to save the schedule to flash periodically so that in the event of a crash, last run times will be preserved. You can use a scheduled task to do this.
```python
import schedule

async def save_schedule():
    schedule.save_flash()

schedule.every(30).minutes.do(save_schedule)
```

It is important to note that when using persistence, the scheduled tasks only need to be created and saved to flash once. The easiest way to do this is to write a deployment function that creates the scheduled tasks, this function would be called once when the board is deployed, or when code is updated.
```python
import schedule

async def job():
    print("I'm working...")

def create_schedule():
    schedule.clear()
    
    schedule.every(30).minutes.do(job)
    schedule.every().hour.do(job)
    schedule.every().day.at("10:30").do(job)
    schedule.every().wednesday.at("13:15").do(job)

    schedule.save_flash()
```

Deepsleep support 
-----
One of the key advantages of a persistent scheduler is the ability for the device to sleep or enter low power mode when no tasks are scheduled to run. This is important for devices that operate on battery power.

The library includes a helper function idle_seconds() which can be used to determine how long to sleep for.

```python
import schedule
import machine

sleep_time = schedule.idle_seconds()

machine.deepsleep(int(sleep_time*1000))
```

When using deep sleep functionality there is no need to run a scheduling task in the main event loop. The device will simply boot, run pending tasks and then go back to sleep.
In this situation the simplifed example from above will now look like this.

```python
import schedule
import machine
import uasyncio as asyncio

async def job():
    print("I'm working...")

async def job_with_argument(name):
    print(f"I am {name}")
    
# This function is only run once during device deployment
def create_schedule():
    schedule.clear()
    
    schedule.every(1).minutes.do(job) 
    schedule.every(10).seconds.do(job_with_argument, name="MicroPython")
    
    schedule.save_flash()
    
async def main_loop():
    # Load schedule from flash
    schedule.load_flash()
    
    # Run pending tasks
    await schedule.run_pending()
    
    # Save schedule, this step is important to make sure the last run time is preserved
    schedule.save_flash()
    
    sleep_time = schedule.idle_seconds()
    
    print(f'Going to sleep for { sleep_time } seconds.')

    machine.deepsleep(sleep_time*1000)
    
    
try:
    asyncio.run(main_loop())
except KeyboardInterrupt:
    print('Interrupted')
except Exception as e:
    print("caught")
    print(e)
finally:
    asyncio.new_event_loop()

```

Limitations 
-----
The following are know limitations that need to be addressed in future releases.

Coros names and arguments are saved to flash and run using eval. This creates two limitations.

1. Only in scope coros that exist inside main can be used
2. There is a potential security risk if the schedule.json file is modified to run arbitary code


----

Patrick Joy `patrick@joytech.com.au`

Inspired by Daniel Bader and distributed under the MIT license. See `LICENSE.txt <https://github.com/ThinkTransit/schedule-micropython/blob/master/LICENSE.txt>` for more information.

https://github.com/ThinkTransit/schedule-micropython
