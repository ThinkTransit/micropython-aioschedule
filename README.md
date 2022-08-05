# micropython-aioschedule

This project is very much a work in progress. Feel free to submit issues/PR's

The concept is to have a simple micropython scheduler that is asynchronous, persistent (tasks are stored in flash and can be reloaded after a reboot/restart).

Based on the python schedule library by dbader https://github.com/dbader/schedule

Python job scheduling for humans. Run Python functions (or any other callable) periodically using a friendly syntax.

- Asycnronous
- A simple to use API for scheduling jobs, made for humans.
- In-process scheduler for periodic jobs. No extra processes needed!
- Persistence, tasks survive reboot/restart
- Tested on MicroPython 1.19

Usage
-----

Requires datetime and functools from micropython-lib

```
Copy schedule.py to your micropython lib directory
```

```python

import schedule
import time
import machine
import uasyncio as asyncio

async def job():
    print("I'm working...")

schedule.every(10).seconds.do(job)
schedule.every(10).minutes.do(job)
schedule.every().hour.do(job)
schedule.every().day.at("10:30").do(job)
schedule.every(5).to(10).minutes.do(job)
schedule.every().monday.do(job)
schedule.every().wednesday.at("13:15").do(job)
schedule.every().minute.at(":17").do(job)

async def job_with_argument(name):
    print(f"I am {name}")

schedule.every(10).seconds.do(job_with_argument, name="Peter")

async def main_loop():
    # Run all outstanding tasks
    # Schedule will yield after all tasks have been run
    await schedule.run_pending()
    
    # Run any other user code here
    
    # Save schedule to flash
    schedule.save_flash()
    
    # Go to sleep until the next task is due
    idle_secs = schedule.idle_seconds()
    logger.debug(f'Going to sleep for { idle_secs } seconds')
    machine.deepsleep(int(idle_secs) * 1000)
    
# Load schedule from flash
schedule.load_flash()

try:
    asyncio.run(main_loop())
except KeyboardInterrupt:
    print('Interrupted')
except Exception as e:
    print("caught")
    print_exception(e)
finally:
    asyncio.new_event_loop()

```
```python
# Save schedule to flash
schedule.save_flash()

# Load schedule from flash
schedule.load_flash()
```

Documentation
-------------

TODO


Meta
----

Patrick Joy `patrick@joytech.com.au`

Inspired by Daniel Bader - `@dbader_org <https://twitter.com/dbader_org>`_ - mail@dbader.org,
`Adam Wiggins' <https://github.com/adamwiggins>`_ article `"Rethinking Cron" <https://adam.herokuapp.com/past/2010/4/13/rethinking_cron/>`_
and the `clockwork <https://github.com/Rykian/clockwork>`_ Ruby module.

Distributed under the MIT license. See `LICENSE.txt <https://github.com/ThinkTransit/schedule-micropython/blob/master/LICENSE.txt>`_ for more information.

https://github.com/ThinkTransit/schedule-micropython
