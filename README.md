# 🎮 Lobby: A Social Platform for Gamers

Profiles, friends, realtime chat, skill-based matchmaking and peer-to-peer voice, all in one repo with one install and no build step.

![python](https://img.shields.io/badge/python-3.10%2B-blue) ![flask](https://img.shields.io/badge/flask-3.x-lightgrey) ![sqlite](https://img.shields.io/badge/db-sqlite-green) ![license](https://img.shields.io/badge/license-MIT-orange)

---

## ✨ Features

- **Accounts & profiles**: register, log in, set a gamertag, bio, favorite games and platforms
- **Friends**: send / accept requests, see who is online
- **Realtime chat**: global lobby, game channels and direct messages over WebSockets
- **Matchmaking**: queue up for a game and get paired by skill rating and preferences
- **Voice chat**: browser-native WebRTC, peer-to-peer, no media server required
- **Zero build tooling**: plain HTML, CSS and vanilla JS, deployable anywhere

---

## 🧱 Tech Stack

| Layer      | Choice                              |
|------------|-------------------------------------|
| Backend    | Flask                               |
| Realtime   | Flask-SocketIO                      |
| Database   | SQLite (via SQLAlchemy)             |
| Voice      | WebRTC (signaling over Socket.IO)   |
| Frontend   | Vanilla HTML / CSS / JavaScript     |

---

## 🚀 Quick Start

```bash
# 1. Clone
git clone https://github.com/<you>/lobby.git
cd lobby

# 2. Create a virtual environment
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure
cp .env.example .env            # then edit SECRET_KEY

# 5. Initialize the database
flask --app app init-db

# 6. Run
python run.py
```

Open **http://localhost:5000**. Open a second browser or an incognito window to test chat, matchmaking and voice between two users.

---

## ⚙️ Configuration

Set these in `.env`:

| Variable        | Default                  | Description                          |
|-----------------|--------------------------|--------------------------------------|
| `SECRET_KEY`    | none (required)          | Session signing key                  |
| `DATABASE_URL`  | `sqlite:///lobby.db`     | Database connection string           |
| `PORT`          | `5000`                   | Server port                          |
| `STUN_URL`      | `stun:stun.l.google.com:19302` | STUN server for WebRTC         |
| `TURN_URL`      | empty                    | Optional TURN server (see Voice)     |
| `TURN_USER` / `TURN_PASS` | empty          | TURN credentials                     |

---

## 🗂 Project Structure

```
lobby/
├── app/
│   ├── __init__.py          # app factory, extensions
│   ├── models.py            # User, Friendship, Message, MatchTicket
│   ├── auth.py              # register / login / logout
│   ├── api.py               # REST endpoints (profiles, friends, history)
│   ├── sockets/
│   │   ├── chat.py          # rooms, DMs, presence
│   │   ├── matchmaking.py   # queue + pairing logic
│   │   └── voice.py         # WebRTC signaling relay
│   ├── static/
│   │   ├── css/
│   │   └── js/
│   │       ├── chat.js
│   │       ├── matchmaking.js
│   │       └── voice.js
│   └── templates/
├── run.py
├── requirements.txt
├── .env.example
└── README.md
```

Each realtime feature lives in its own module, so you can add or remove one without touching the others.

---

## 🔌 Realtime Events

### Chat

| Event            | Direction        | Payload                     |
|------------------|------------------|-----------------------------|
| `chat:join`      | client → server  | `{ room }`                  |
| `chat:message`   | both             | `{ room, text }`            |
| `chat:dm`        | both             | `{ to, text }`              |
| `presence:update`| server → client  | `{ user, status }`          |

### Matchmaking

| Event            | Direction        | Payload                     |
|------------------|------------------|-----------------------------|
| `mm:queue`       | client → server  | `{ game, mode }`            |
| `mm:cancel`      | client → server  | none                        |
| `mm:matched`     | server → client  | `{ match_id, players }`     |

### Voice signaling

| Event            | Direction        | Payload                     |
|------------------|------------------|-----------------------------|
| `voice:join`     | client → server  | `{ room }`                  |
| `voice:offer`    | relayed          | `{ to, sdp }`               |
| `voice:answer`   | relayed          | `{ to, sdp }`               |
| `voice:ice`      | relayed          | `{ to, candidate }`         |

---

## 🎯 How Matchmaking Works

1. A player emits `mm:queue` with their game and mode.
2. The server stores a ticket with the player's skill rating and queue time.
3. A background task scans the queue and pairs tickets whose ratings fall within an allowed window.
4. The window **widens the longer a ticket waits**, so nobody is stuck forever.
5. Matched players receive `mm:matched` and are placed in a private room with chat and voice.

Tune the rating window and growth rate in `app/sockets/matchmaking.py`.

---

## 🎙 Voice Chat Notes

Voice is peer-to-peer. The server only relays signaling messages (offer, answer, ICE candidates) and never touches audio.

- **Works out of the box** on most home networks using a public STUN server.
- **Strict NATs / corporate networks** need a TURN server. Set `TURN_URL`, `TURN_USER` and `TURN_PASS`. [coturn](https://github.com/coturn/coturn) is a solid self-hosted option.
- Browsers require **HTTPS** for microphone access everywhere except `localhost`.
- Pure mesh P2P is great for small groups (about 2 to 5 people). Larger rooms need an SFU.

---

## 🧩 Extending

- **New game channel**: add a row to the games list and the chat rooms follow automatically.
- **New realtime feature**: create a module in `app/sockets/`, register its handlers in `app/__init__.py`.
- **Swap the database**: change `DATABASE_URL` to Postgres or MySQL; models are plain SQLAlchemy.
- **Scale out**: add Redis as the Socket.IO message queue and run multiple workers.

---

## 🛣 Roadmap

- [ ] Parties and persistent group voice rooms
- [ ] Rating updates after reported match results
- [ ] Image and clip sharing in chat
- [ ] Moderation tools (mute, block, report)
- [ ] OAuth login (Discord, Steam)
- [ ] Docker Compose setup

---

## 🛠 Troubleshooting

| Problem                          | Fix                                                           |
|----------------------------------|---------------------------------------------------------------|
| Chat connects then drops         | Install `eventlet` or `gevent` and run via `run.py`           |
| Mic permission denied            | Use `localhost` or serve over HTTPS                           |
| Voice connects but no audio      | Likely NAT issue, configure a TURN server                     |
| `database is locked`             | Avoid multiple writers on SQLite, move to Postgres for prod   |

---

## 🤝 Contributing

1. Fork the repo and create a branch: `git checkout -b feature/my-feature`
2. Commit your changes and open a pull request
3. Keep modules small and focused

---

## 📄 License

MIT. See [LICENSE](LICENSE).
