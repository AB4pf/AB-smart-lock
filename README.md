# Smart Locker Rental Kiosk

**2nd Place Overall, CSC 132 Project Showcase, Louisiana Tech University (Fall 2026)**

A self-service locker kiosk that runs on a Raspberry Pi. Users pick a locker, choose how long they need it, tap an NFC card to pay, and get a PIN to open, extend, or end their rental. One locker is wired to a real electronic lock and door sensor; the others are simulated.


## Features

- **Rent a locker:** choose an available locker, enter name, phone, and a 4–6 digit PIN, pick a duration, tap a card to pay
- **5 pricing tiers:** 30 min ($2), 1 hour ($3), 2 hours ($5), 4 hours ($8), all day ($12)
- **Manage a rental:** log in with phone + PIN to open the locker, extend the time, or end the rental
- **Real hardware control:** NFC card reader, relay-driven electronic lock, and a door sensor that reports open/closed in real time
- **Admin panel:** PIN-protected screen to view all lockers, unlock any locker, end rentals, and disable or enable lockers
- **Security:** PINs hashed with Werkzeug (never stored in plain text); 5 wrong PINs lock that phone number out for 5 minutes; admin PIN read from an environment variable
- **Simulation mode:** on a laptop, the hardware layer runs in "fake mode" so the full app can be developed and tested without a Pi

## Tech stack

| Layer | Technology |
| --- | --- |
| Back end | Python, Flask (16 REST endpoints) |
| Database | SQLite |
| Front end | HTML, CSS, JavaScript (single-page kiosk with 14 screens) |
| Hardware | Raspberry Pi, PN532 NFC reader (I2C), relay module, magnetic door sensor, RPi.GPIO |

## Architecture

```
Browser (kiosk UI)  --->  Flask API (app.py)  --->  SQLite (locker.db)
                                 |
                                 v
                          hardware.py  --->  NFC reader / relay lock / door sensor
                       (fake mode on a laptop)
```

## Run it

```bash
python3 -m venv venv
source venv/bin/activate
pip install flask
python3 init_db.py      # create the database
python3 seed_db.py      # add 4 lockers
ADMIN_PIN=<choose-a-pin> python3 app.py
```

Open `http://127.0.0.1:5000`. On a laptop the hardware runs in simulation mode automatically.

On the Raspberry Pi, also install `RPi.GPIO`, `adafruit-blinka`, and `adafruit-circuitpython-pn532`, and enable I2C.

## API overview

| Method | Endpoint | Purpose |
| --- | --- | --- |
| GET | `/lockers` | List lockers and their status |
| POST | `/scan-card` | Wait for an NFC card tap |
| POST | `/rent` | Start a rental |
| POST | `/check-rental` | Log in with phone + PIN |
| POST | `/open-locker` | Unlock the user's locker |
| POST | `/extend-rental` | Add time |
| POST | `/end-rental` | End the rental and free the locker |
| POST | `/door-status` | Door sensor update |
| POST | `/forgot-pin` | Directs the user to an administrator |
| GET/POST | `/admin/*` | Admin: view, unlock, end, disable, enable |

## Testing

The `test_*.py` scripts send requests to each endpoint (`test_rent.py`, `test_open.py`, `test_extend.py`, `test_end.py`, `test_admin.py`) and check each hardware part on the Pi (`test_nfc.py`, `test_lock.py`, `test_door.py`, `test_hardware.py`).

## My role

Team of 3. I built the Flask back end, the kiosk front end, and the Raspberry Pi hardware integration. Teammates designed the database schema and wrote the hardware tests.

## What I'd improve next

- Serve over HTTPS and send the admin PIN in the request body or a session, not the URL
- Connect a real payment provider (card taps are simulated)
- Send SMS reminders before a rental expires
