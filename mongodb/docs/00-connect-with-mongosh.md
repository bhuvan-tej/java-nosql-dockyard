# 00 · Connect to Atlas with mongosh

All lessons are run from the command line with **mongosh** (the MongoDB Shell).

## 1. Install

```bash
brew install mongosh        # macOS
mongosh --version
```

Other platforms: <https://www.mongodb.com/try/download/shell>

## 2. Allow your IP in Atlas

Go to **Atlas → Security → Network Access → Add IP Address → Add Current IP Address**.
Without this, the connection times out.

## 3. Get the connection string

Go to **Atlas → Database → Connect → Shell**. It looks like:

```
mongodb+srv://<cluster>.xxxxx.mongodb.net/
```

## 4. Connect

```bash
mongosh "mongodb+srv://<cluster>.xxxxx.mongodb.net/" --username <user>
# you'll be prompted for the password, so it isn't saved in your shell history
```

Optional: store the URI once in `~/.zshrc` (without the password) so you don't have to retype it:

```bash
export MONGODB_URI="mongodb+srv://<cluster>.xxxxx.mongodb.net/"
mongosh "$MONGODB_URI" --username <user>
```

Never commit a URI that contains a password.

## 5. Connection check

Inside mongosh:

```js
db.runCommand({ ping: 1 })         // { ok: 1 }  -> connected
db.version()                       // server version
show dbs                           // databases you can see
use dockyard                       // switch to (or create) the database used in these lessons
db.getName()                       // dockyard
show collections                   // empty until the first insert
```

`use dockyard` doesn't create anything yet. The database appears after the first write.

## 6. Running queries

| Way | How |
|---|---|
| Interactive | Type or paste queries at the `dockyard>` prompt |
| One-liner | `mongosh "$MONGODB_URI" --username <user> --eval 'db.getSiblingDB("dockyard").users.countDocuments()'` |
| Script file | `mongosh "$MONGODB_URI" --username <user> --file practice.js` |
| Load inside a session | `load("practice.js")` |

## 7. Handy shell commands

```js
help                         // general help
db.users.help()              // collection methods
cls                          // clear screen
it                           // show the next batch of find() results
exit                         // quit
```

## 8. Troubleshooting

| Error | Fix |
|---|---|
| `connection timed out` / `Server selection timed out` | Your IP isn't in Network Access |
| `Authentication failed` | Wrong user or password. This must be a **Database Access** user, not your Atlas login. |
| `querySrv ENOTFOUND` | Typo in the cluster host, or DNS/VPN is blocking SRV lookups |
| Special characters in the password inside the URI | URL-encode them (`@` becomes `%40`), or use `--username` and type it at the prompt |
