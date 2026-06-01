

```
================================================================
                  PYTHON ASYNCIO - COMPLETE NOTES
================================================================


================================================================
 1. WHAT IS ASYNCIO?
================================================================

asyncio = Python's built-in library for writing concurrent code
          using async/await syntax.

Core Idea:
----------
→ Lets a SINGLE thread juggle many tasks that spend most of
  their time WAITING (network, file I/O, DB, timers).
→ While one task waits, another runs.
→ Cooperative multitasking - tasks voluntarily yield control
  at await points.

Simple Analogy (Cooking Dinner):
--------------------------------
  Without async:  Boil pasta (10m) → Bake bread (15m) → Salad (5m)
                  Total = 30 minutes  ✗
  
  With async:     Start pasta + Start bread + Make salad together
                  Total = 15 minutes  ✓

Key Point:
----------
→ asyncio = NOT wasting time during waiting.
→ Same single worker, but smarter scheduling.


================================================================
 2. def  vs  async def
================================================================

+------------------+----------------------+----------------------+
| Feature          | def (normal)         | async def (async)    |
+------------------+----------------------+----------------------+
| Runs when called?| Yes, immediately     | No, returns coroutine|
| Use await inside?| ✗ No                 | ✓ Yes                |
| Can pause/resume?| ✗ No                 | ✓ Yes                |
| How to run?      | func()               | await func()  OR     |
|                  |                      | asyncio.run(func())  |
+------------------+----------------------+----------------------+

Example:
--------
  def greet():
      return "Done"
  
  result = greet()        → Runs immediately, result = "Done"


  async def greet():
      return "Done"
  
  result = greet()        → Does NOT run!
                            result = <coroutine object>
  
  result = asyncio.run(greet())   → NOW runs, result = "Done"


Key Intuition:
--------------
→ def        = worker who starts the job immediately
→ async def  = worker who takes order but waits for "go" signal
               (await / asyncio.run)


================================================================
 3. WHY DO WE NEED  await ?
================================================================

await tells Python:
-------------------
  "Pause here. Let other tasks run while we wait.
   Come back when my thing is ready."

Without await:
--------------
→ async functions behave like normal ones
→ No pause points = no benefit from asyncio
→ Calling async function without await = NOTHING happens
  (just creates a useless coroutine object + warning)

Real-life Analogy (Restaurant):
-------------------------------
  Without await:  Order pizza → stand at counter staring
                  → Order drink → stand → Order dessert → stand
                  (Blocking everyone behind you ✗)
  
  With await:     Order pizza → await (sit down, others order)
                  → kitchen says ready → continue
                  (Kitchen serves many people ✓)

Code Example:
-------------
  async def make_coffee():
      print("Coffee started")
      await asyncio.sleep(3)        ← pause point
      print("Coffee ready")
  
  async def make_toast():
      print("Toast started")
      await asyncio.sleep(2)        ← pause point
      print("Toast ready")
  
  async def main():
      await asyncio.gather(make_coffee(), make_toast())
  
  asyncio.run(main())

  Output:
    Coffee started
    Toast started
    Toast ready    (after 2s)
    Coffee ready   (after 3s)
  
  Total = 3 seconds, NOT 5 seconds  ✓


What if you forget await?
-------------------------
  async def make_coffee():
      asyncio.sleep(3)              ← ✗ No await!
  
  → Warning: "coroutine was never awaited"
  → sleep never actually happens


================================================================
 4.  asyncio.create_task()  - RUNNING IN BACKGROUND
================================================================

Purpose:
--------
→ Start a task in the BACKGROUND and keep going.
→ Don't wait for it right now.
→ Like pressing "START" on the microwave and walking away.

Syntax:
-------
  task = asyncio.create_task(some_async_func())
  # ... do other work ...
  await task    ← collect the result when you need it

+----------------------------+--------------------------------+
| Code                       | What happens                   |
+----------------------------+--------------------------------+
| await func()               | Start AND wait here            |
| asyncio.create_task(func())| Start in background, keep going|
+----------------------------+--------------------------------+

Example:
--------
  async def main():
      # Press start on microwave
      task = asyncio.create_task(make_coffee())
      
      # Don't wait - do other stuff
      print("Buttering toast...")
      print("Setting the table...")
      
      # NOW I need the coffee
      await task
      print("Drinking coffee!")


Three Versions Compared:
------------------------
  ✗ Slow (sequential):
      await make_coffee()    # 3s
      await make_toast()     # 2s
      → Total: 5 seconds
  
  ✓ Fast (create_task):
      coffee = asyncio.create_task(make_coffee())
      toast  = asyncio.create_task(make_toast())
      await coffee
      await toast
      → Total: 3 seconds
  
  ✓ Fast (shortcut with gather):
      await asyncio.gather(make_coffee(), make_toast())
      → Total: 3 seconds


When to use what?
-----------------
→ gather       = start several tasks together, wait for ALL
                 (simple and clean)
→ create_task  = start a task, do OTHER work in between,
                 then wait later (more flexible)


================================================================
 5.  asyncio.run()  - ENTRY POINT
================================================================

What it is:
-----------
→ A NORMAL function (not async).
→ The BRIDGE from normal Python → async world.
→ Used ONCE at the top level.

Why no await before asyncio.run()?
----------------------------------
→ asyncio.run() handles all the awaiting INTERNALLY.
→ await only works INSIDE async def functions.
→ asyncio.run() is called from NORMAL code → no await needed.

The Two Worlds:
---------------
  +---------------------------+
  |  🌐 NORMAL WORLD          |
  |  - Can't use await        |
  |  - Use asyncio.run() to   |
  |    enter async world      |
  +---------------------------+
              │
              │ asyncio.run()  ← the DOOR
              ▼
  +---------------------------+
  |  ⚡ ASYNC WORLD            |
  |  - Use await              |
  |  - Use create_task        |
  |  - Use gather             |
  |  - Use asyncio.sleep      |
  +---------------------------+

Analogy (Swimming Pool):
------------------------
  Normal code   = standing on land
  Async code    = swimming in pool
  asyncio.run() = jumping into pool (done from land)
  await         = swimming strokes (only inside pool)


================================================================
 6. THE COMPLETE RULES - WHERE TO USE WHAT
================================================================

+----------------------------+---------------------------------+
| Calling what?              | How to call                     |
+----------------------------+---------------------------------+
| Normal function (anywhere) | normal_func()                   |
| Async func (inside async)  | await async_func()              |
| Async func (top-level)     | asyncio.run(async_func())       |
| Async func (background)    | asyncio.create_task(func())     |
+----------------------------+---------------------------------+

Common Mistakes:
----------------
  ✗ await normal_func()         → Can't await a normal function
  ✗ asyncio.run() inside async  → Will crash
  ✗ await at top-level          → SyntaxError
  ✗ Calling async func without
    await/create_task/run       → Nothing happens + warning

Correct Patterns:
-----------------
  ✓ async def main():
        normal_func()              # call normal as usual
        await async_func()         # await for async
        task = asyncio.create_task(other())
        await task
    
    asyncio.run(main())            # entry point


================================================================
 7. WHEN TO USE ASYNC - USE CASES
================================================================

The Golden Rule:
----------------
  Async = good for I/O-bound work
        = bad for CPU-bound work

I/O-bound  = waiting for external stuff (network, disk, DB)
CPU-bound  = doing heavy computation (math, ML, image processing)


✓ USE ASYNC FOR:
-----------------
  • Calling many APIs at once
  • Web scraping (many pages in parallel)
  • Web servers (FastAPI, aiohttp)
  • Chat apps / WebSockets / real-time apps
  • Database-heavy apps (asyncpg, motor)
  • Microservices that call other services
  • Crawlers, bots, monitoring tools


✗ DO NOT USE ASYNC FOR:
-----------------------
  • Image / video processing      → use multiprocessing
  • Machine learning training     → use multiprocessing
  • Heavy math / calculations     → use multiprocessing
  • Simple scripts (no waiting)   → use normal code
  • Single API call               → no benefit


Decision Flowchart:
-------------------
  Does code WAIT for slow things (network/file/DB)?
   │
   ├── No  → Use normal code
   │
   └── Yes → Do you do MANY of them?
            │
            ├── No  → Normal code is fine
            │
            └── Yes → Use ASYNC ✓


Speed Comparison Example (Weather for 10 cities):
-------------------------------------------------
  Without async:
    for city in cities:
        weather = get_weather(city)    # 1s each
    → Total: 10 seconds ✗
  
  With async:
    results = await asyncio.gather(
        *[get_weather(c) for c in cities]
    )
    → Total: ~1 second ✓


================================================================
 8. KEY INTUITION SUMMARY
================================================================

9. async def  →  makes a function PAUSABLE
                 (returns coroutine, doesn't run immediately)

10. await      →  the PAUSE POINT
                 "wait here, let other tasks run meanwhile"
                 (only works inside async def)

11. asyncio.run() →  the ENTRY DOOR from normal → async
                    (call once at top level, no await)

12. create_task() →  start in BACKGROUND, await later
                    (for running things in parallel with
                     other work in between)

13. gather()  →  shortcut to run MANY tasks together
                and wait for all of them

14. Without await:  async functions DON'T RUN
                   (calling them = useless coroutine object)

15. async helps WAITING, not WORKING
                   (I/O-bound = ✓, CPU-bound = ✗)


================================================================
 9. QUICK CHEAT SHEET
================================================================

  import asyncio
  
  # Define async function
  async def my_func():
      await asyncio.sleep(1)
      return "done"
  
  # Run from normal code
  asyncio.run(my_func())
  
  # Inside another async function:
  async def main():
      # Wait for it
      result = await my_func()
      
      # OR run in background
      task = asyncio.create_task(my_func())
      # ...do other stuff...
      result = await task
      
      # OR run many in parallel
      results = await asyncio.gather(
          my_func(),
          my_func(),
          my_func()
      )
  
  asyncio.run(main())


================================================================
                        END OF NOTES
================================================================
```