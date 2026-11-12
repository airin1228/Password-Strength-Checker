# Password Strength Checker

A web app that tells you how strong a password is, explains **why**, and shows **exactly what to change** to make it stronger.

- The **analysis** is written in **Python** (`password_strength_checker.py`).
- The **interface** is **HTML, CSS and JavaScript** (`templates/index.html`).
- **Flask** (a small Python web framework) connects the two, so the page you see in the browser gets its results from the Python code.

---

## Table of contents

1. [How it works (big picture)](#1-how-it-works-big-picture)
2. [Features explained](#2-features-explained)
3. [Project structure](#3-project-structure)
4. [Requirements](#4-requirements)
5. [Installation and running](#5-installation-and-running)
6. [Using the web interface](#6-using-the-web-interface)
7. [Using the command-line version](#7-using-the-command-line-version)
8. [How the checking works](#8-how-the-checking-works)
9. [The password generator](#9-the-password-generator)
10. [API reference](#10-api-reference)
11. [Privacy and security](#11-privacy-and-security)
12. [Troubleshooting](#12-troubleshooting)
13. [Limitations](#13-limitations)
14. [Customizing the project](#14-customizing-the-project)
15. [FAQ](#15-faq)

---

## 1. How it works (big picture)

```
 You type a password in the browser
            |
            v
 index.html (JavaScript) waits 120 ms after you stop typing,
 then sends the password to the Python server
            |
            v
 app.py (Flask) receives it at /api/analyze
            |
            v
 password_strength_checker.py runs the checks
 (length, character types, patterns, entropy, crack time)
            |
            v
 Flask sends the results back as JSON
            |
            v
 index.html updates the meter, stats, checklist and tips
```

Everything happens on your own computer. The "server" is just a Python program running on your machine at the address `127.0.0.1`, which only you can reach.

---

## 2. Features explained

| Feature | What it does | Why it is useful |
|---|---|---|
| **Length check** | Checks if the password has at least 12 characters. | Length is the single biggest factor in strength. Every extra character multiplies the number of guesses an attacker needs. |
| **Character variety** | Checks for lowercase letters, uppercase letters, numbers and special characters. | Mixing character types makes the pool of possible characters larger, so guessing is harder. |
| **Pattern detection** | Flags common passwords, repeated characters and sequences like `1234` or `qwerty`. | Attackers try these predictable passwords first, so they are weak even if they look "complex". |
| **Strength classification** | Gives a score from 0 to 5 and a label from Very Weak to Very Strong. | Gives you a quick, easy-to-read verdict. |
| **Entropy estimate** | Shows how unpredictable the password is, in bits. | A more precise number than the label. Higher is better. |
| **Crack-time estimate** | Shows roughly how long a brute-force attack would take. | Makes the risk easier to understand than a bit count. |
| **Improvement tips** | Lists exactly what is missing, such as "Add uppercase letters". | You know what to fix instead of guessing. |
| **Live feedback** | Results update as you type. | You can see the effect of each change immediately. |
| **Password generator** | Creates a random, strong password of your chosen length. | Random passwords are much stronger than ones people invent. |
| **Command-line mode** | Runs the same checks in a terminal without Flask. | Useful for quick checks or for learning how the logic works. |

---

## 3. Project structure

```
password_checker_app/
├── app.py                         # Flask server and API routes
├── password_strength_checker.py   # Core logic (also a CLI)
├── requirements.txt               # Python dependencies
├── templates/
│   └── index.html                 # Frontend page
└── README.md                      # Documentation
```

What each file does:

- **`app.py`**: The Flask server. It defines three routes: the home page (`/`), the analysis route (`/api/analyze`) and the generator route (`/api/generate`). It does no checking itself. It calls the functions in `password_strength_checker.py` and returns their results.
- **`password_strength_checker.py`**: The core logic. It contains `analyze()` (runs all checks), `classify()` (turns a score into a label), `crack_time()` (turns entropy into a readable time), `generate_password()` (makes a random password) and the command-line interface.
- **`requirements.txt`**: Lists the Python packages the project needs (only Flask).
- **`templates/index.html`**: The page you see in the browser. It contains the layout, the styling (CSS) and the JavaScript that talks to the server. Flask looks for HTML pages in a folder named exactly `templates`, so do not rename it.
- **`README.md`**: This file.

---

## 4. Requirements

- **Python 3.8 or newer.** Check with `python --version`.
- **Flask 3.0 or newer.** Installed in the next section.
- **A web browser**, such as Chrome, Edge or Firefox.
- **Internet access is optional.** It is only used to load the Google Fonts (Space Grotesk, Inter, JetBrains Mono). Without internet, the page still works and falls back to your system fonts.

---

## 5. Installation and running

### Windows

**Step 1: Install Python (if you do not have it).**
Download it from python.org. On the first screen of the installer, tick **"Add Python to PATH"** before clicking Install. Without this, the commands below will not be recognized.

**Step 2: Open Command Prompt in the project folder.**
Unzip the project, open the `password_checker_app` folder in File Explorer, click the address bar, type `cmd` and press Enter.

**Step 3 (optional but recommended): Create a virtual environment.**
This keeps the project's packages separate from the rest of your computer.
```
python -m venv venv
venv\Scripts\activate
```
After activating, you will see `(venv)` at the start of the line.

**Step 4: Install the dependencies.**
```
pip install -r requirements.txt
```
This reads `requirements.txt` and installs Flask.

**Step 5: Start the server.**
```
python app.py
```
You should see a message saying the app is running on `http://127.0.0.1:5000`.

**Step 6: Open the app.**
Go to **http://127.0.0.1:5000** in your browser.

**Step 7: Stop the server.**
Click the Command Prompt window and press `Ctrl+C`.

> If `python` or `pip` is not recognized, use `py` and `py -m pip` instead (for example `py -m pip install -r requirements.txt` and `py app.py`).

### macOS / Linux

```
python3 -m venv venv
source venv/bin/activate
pip3 install -r requirements.txt
python3 app.py
```
Then open http://127.0.0.1:5000.

---

## 6. Using the web interface

Here is every element on the page and what it does.

### Password box
Type or paste a password. It is hidden by default (shown as dots).
- **Show / Hide**: switches between dots and readable text.
- **Copy**: copies the password to your clipboard. The button briefly says "Copied" to confirm.

### Strength meter
Five bars that fill up as the score rises.
- **Red**: score 1 or 2 (weak).
- **Amber**: score 3 (fair).
- **Green**: score 4 or 5 (strong).

Below the bars you see the **label** (for example "Strong") and the **score** (for example "4 / 5").

### Stat cards
- **Length**: how many characters you typed.
- **Entropy**: the unpredictability of the password in bits.
- **Time to crack**: the estimated time a brute-force attack would need.

### Checklist
Six items that get a green tick when they pass:
1. At least 12 characters
2. Lowercase letter
3. Uppercase letter
4. Number
5. Special character (anything that is not a letter or digit, such as `! @ # $`)
6. No obvious pattern

### Notes area
- **Red boxes** are serious problems, such as "Contains a commonly leaked password".
- **Amber boxes** are improvement tips, such as "Add numbers (0-9)".
- **Green box** appears when every check passes.

### Password generator panel
- **Slider**: choose a length from 12 to 40 characters.
- **Generate**: creates a new random password and shows it in the box.
- **Check this one**: puts the generated password into the main box and analyzes it.
- **Copy**: copies the generated password.

---

## 7. Using the command-line version

You can use the checker without the web page and without Flask.

```
python password_strength_checker.py            # hidden input
python password_strength_checker.py --show     # visible input
python password_strength_checker.py --generate # generate a password and rate it
```

- **Hidden input** (default): nothing appears as you type. This is normal and is done so no one can read the password over your shoulder.
- **`--show`**: your typing is visible.
- **`--generate`**: prints a random 16-character password, then shows its report.

After each report, it asks `Check another? (y/n)`. Type `y` to continue or anything else to quit.

### Example output

```
==============================================
 Strength : Very Weak  [-----] 0/5
 Entropy  : ~57 bits
 Crack time (estimate): 167 days
==============================================
 Checks:
   [ ] At least 12 characters
   [x] Lowercase letter
   [ ] Uppercase letter
   [x] Number
   [ ] Special character
   [ ] No obvious pattern

 Problems found:
   ! Contains a commonly leaked password.

 How to improve:
   - Use at least 12 characters (longer is better).
   - Add uppercase letters (A-Z).
   - Add special characters (e.g. ! @ # $ % &).
```

How to read it:
- The **bar** `[-----]` shows the score visually (`#` is a filled point).
- **`[x]`** means the check passed, **`[ ]`** means it failed.
- **Problems found** lists serious weaknesses. **How to improve** lists what to add.

---

## 8. How the checking works

### 8.1 The five basic checks

| Check | Rule |
|---|---|
| Length | 12 or more characters |
| Lowercase | At least one letter from `a` to `z` |
| Uppercase | At least one letter from `A` to `Z` |
| Number | At least one digit from `0` to `9` |
| Special | At least one character that is not a letter or digit |

### 8.2 Pattern detection

Even a password that passes every basic check can be weak if it is predictable. Four rules look for this:

| Rule | What triggers it | Example |
|---|---|---|
| Too short | Fewer than 8 characters | `Ab1!` |
| Common password | Contains any word from the built-in list of leaked passwords (case-insensitive) | `Password123!`, `MyAdmin99` |
| Repeated characters | The same character 3 or more times in a row | `Heeey!!!` |
| Sequence or keyboard run | Runs such as `0123`, `1234`, `abcd`, `qwerty`, `asdf` | `Abcd1234!` |

If **any** of these triggers, the "No obvious pattern" check fails and the score is reduced (see 8.3).

Note that the common-password check looks for the word **anywhere inside** the password, not just an exact match. That is why `Password123!` is flagged even though it looks complex.

### 8.3 The score (0 to 5)

The score starts at 0 and gains points like this:

| Points | Condition |
|---|---|
| +1 | At least 8 characters |
| +1 | At least 12 characters |
| +1 | Has both lowercase **and** uppercase letters |
| +1 | Has a number |
| +1 | Has a special character |
| **-2** | A weak pattern was detected (the score never goes below 0) |

The score is then turned into a label:

| Score | Label | Meaning |
|---|---|---|
| 0 or 1 | Very Weak | Easy to guess. Do not use. |
| 2 | Weak | Likely to be cracked quickly. |
| 3 | Fair | Acceptable only for low-value accounts. |
| 4 | Strong | Good for most uses. |
| 5 | Very Strong | Meets every recommended rule. |

**Worked examples**

- `password123`: 11 characters (+1 for 8+), lowercase only, has a number (+1) = 2. A common password is found (-2) = **0, Very Weak**.
- `Hello2024`: 9 characters (+1), both cases (+1), number (+1) = 3. No pattern problems = **3, Fair**.
- `Kp7#vR2!mX9qLz4@`: 16 characters (+2), both cases (+1), number (+1), special (+1) = 5 = **Very Strong**.

### 8.4 Entropy

Entropy measures how many guesses an attacker would need, expressed in **bits**. Each extra bit doubles the number of possible passwords.

**Formula:**
```
entropy = length x log2(size of the character pool)
```

The **pool size** is the total number of possible characters, adding up the types your password uses:

| Type present | Adds to pool |
|---|---|
| Lowercase | 26 |
| Uppercase | 26 |
| Digits | 10 |
| Special characters | 32 |

Example: `password123` uses lowercase and digits, so the pool is 36. Its length is 11, so entropy = 11 x log2(36) = about **57 bits**.

As a rough guide: under 40 bits is weak, 60 to 80 bits is decent, and 100+ bits is very strong.

This formula assumes every character was chosen randomly. A human-made password is more predictable than the number suggests (see [Limitations](#13-limitations)).

### 8.5 Crack-time estimate

```
seconds = 2^entropy / 10,000,000,000
```

This assumes an attacker making **10 billion guesses per second**, which is realistic for an offline attack with modern graphics cards against a fast hash. The result is converted to the largest sensible unit (seconds, minutes, hours, days, years or centuries).

- Under 1 second is shown as "instantly".
- Anything beyond 1000 centuries is shown as "trillions of years". Treat this as a catch-all meaning "effectively uncrackable by brute force".

Real attacks against well-protected systems (with rate limiting or slow hashing) are much slower, so this is a worst-case figure.

---

## 9. The password generator

The generator creates random passwords using Python's **`secrets`** module, which is designed for security. (Python's regular `random` module is not safe for passwords because its output can be predicted.)

How it builds a password:

1. It picks **one lowercase, one uppercase, one digit and one symbol**, guaranteeing all four types are present.
2. It fills the rest of the length with random characters from all four sets.
3. It **shuffles** everything so the guaranteed characters are not always in the first four positions.

Details:
- **Length**: 12 to 40 on the web page (the API accepts up to 64).
- **Ambiguous characters removed**: the letters `l`, `I`, `O` and the digits `0`, `1` are left out so the password is easy to read and type.
- **Symbols used**: `! @ # $ % ^ & * - _ = + ?`

---

## 10. API reference

The web page uses these routes, and you can call them yourself.

### `GET /`
Returns the web page.

### `POST /api/analyze`
Analyzes a password.

**Request body (JSON):**
```json
{ "password": "Kp7#vR2!mX9qLz4@" }
```

**Response (JSON):**
```json
{
  "empty": false,
  "score": 5,
  "level": "Very Strong",
  "entropy": 105,
  "crack_time": "trillions of years",
  "length": 16,
  "checks": {
    "length": true, "lowercase": true, "uppercase": true,
    "number": true, "special": true, "pattern": true
  },
  "issues": [],
  "tips": []
}
```

| Field | Meaning |
|---|---|
| `empty` | `true` if no password was sent (other fields are then absent) |
| `score` | 0 to 5 |
| `level` | The label for the score |
| `entropy` | Estimated bits of entropy |
| `crack_time` | Human-readable estimate |
| `length` | Number of characters |
| `checks` | Pass (`true`) or fail (`false`) for each of the six checks |
| `issues` | Serious problems found |
| `tips` | Suggestions for improvement |

### `GET /api/generate?length=16`
Returns a random password.

```json
{ "password": "W!G!S8hqA+eu7=g6Cx7Q" }
```

`length` is optional (default 16). Values are limited to the range 12 to 64.

---

## 11. Privacy and security

- **Local only.** The server listens on `127.0.0.1`, which means only your own computer can connect to it.
- **No storage, no logging.** Passwords are analyzed in memory and discarded. The app does not write them to a file or database.
- **No third parties.** The only outside connection is the Google Fonts stylesheet, which does not receive your password.
- **Good habit:** even with all of this, do not type a password you actively use into any tool you have not reviewed. Test with a password of similar style instead.
- **Do not put this on the internet as is.** Flask's built-in server is meant for development only. If you ever deploy it publicly, use a production server and HTTPS, otherwise passwords would travel unencrypted.

---

## 12. Troubleshooting

| Problem | Cause and fix |
|---|---|
| `'python' is not recognized` | Python is not on PATH. Reinstall and tick "Add Python to PATH", or use `py` instead of `python`. |
| `ModuleNotFoundError: No module named 'flask'` | Flask is not installed. Run `pip install -r requirements.txt`. If you used a virtual environment, make sure it is activated. |
| `Address already in use` | Something else is using port 5000. Close the other program or change `port=5000` in `app.py`. |
| Page loads but says "Server not reachable" | The server stopped. Check the terminal window and restart with `python app.py`. |
| `TemplateNotFound: index.html` | The `templates` folder is missing or renamed, or you ran `app.py` from a different folder. Run it from inside `password_checker_app`. |
| Fonts look different | You are offline, so Google Fonts could not load. The page still works with fallback fonts. |
| The Copy button says "Failed" | Some browsers block clipboard access. Use `http://127.0.0.1:5000` (not a network address) and allow clipboard permission. |
| No text appears while typing in the command line | This is normal in hidden mode. Use `--show` to see your typing. |

---

## 13. Limitations

- **Entropy assumes randomness.** `Summer2024!` scores well on length and variety, but attackers try patterns like "word + year + symbol" early. Its real strength is far lower than its entropy figure.
- **The common-password list is short** (19 entries). Real attackers use lists with millions of leaked passwords.
- **Substring matching can over-flag.** A password merely containing a word like `master` or `admin` is marked as a problem, even if the rest is random.
- **Crack time is an estimate**, based on one assumed guessing speed.
- **No dictionary, name or date detection.** The checker does not notice birthdays, names or ordinary words unless they are in the common list.
- **Local development server only**, as noted above.

---

## 14. Customizing the project

| What you want to change | Where |
|---|---|
| Add or remove common passwords | `COMMON_PASSWORDS` in `password_strength_checker.py` |
| Change which sequences are flagged | `SEQUENCE_PATTERN` in `password_strength_checker.py` |
| Change how many repeated characters trigger a flag | `REPEAT_PATTERN` (the `{2,}` means 3 or more in a row) |
| Change the recommended length (12) | The `len(password) >= 12` lines and the tip text in `analyze()` |
| Change labels | `LEVELS` in `password_strength_checker.py` |
| Change the assumed guessing speed | `GUESSES_PER_SECOND` in `password_strength_checker.py` |
| Change generator symbols or characters | The strings inside `generate_password()` |
| Change colours | The CSS variables at the top of `templates/index.html` (`--teal`, `--amber`, `--red` and others) |
| Change the port | `port=5000` at the bottom of `app.py` |

After editing any Python file, stop the server (`Ctrl+C`) and start it again. HTML changes only need a browser refresh.

---

## 15. FAQ

**Is a "Very Strong" result a guarantee my password is safe?**
No. It means the password passes every rule in this tool. It does not know whether the password was leaked in a data breach, or whether you reuse it on other sites. Use a unique password for every account and a password manager.

**Why is a long password with only lowercase letters not "Very Strong"?**
The score rewards variety as well as length. Adding digits, capitals and symbols raises the score. In real terms, a long random passphrase can still be very strong, so treat the score as guidance.

**Why did my complex-looking password get flagged?**
Probably because it contains a common word, a repeated run or a sequence such as `1234`. Read the red boxes for the exact reason.

**Can I use the checker without Flask?**
Yes. Run `password_strength_checker.py` directly (see [section 7](#7-using-the-command-line-version)). It only uses Python's standard library.

**Why does the generator leave out some letters and digits?**
`l`, `I`, `O`, `0` and `1` look alike in many fonts, so they are removed to avoid typing mistakes.

**Does it work on a phone?**
The layout adapts to small screens, but the app runs on your computer, so it is meant to be used in a browser on the same machine.
