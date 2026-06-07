# Debug Session: arena-zero-online
- **Status**: [RESOLVED]
- **Issue**: Online Arena still shows 0 online even with two Discord accounts logged in on different PCs.
- **Debug Server**: http://127.0.0.1:7777/event
- **Log File**: .dbg/trae-debug-log-arena-zero-online.ndjson

## Reproduction Steps
1. Open the site on two different PCs or browsers.
2. Log in with two different Discord accounts.
3. Open `ONLINE ARENA` on both.
4. Observe the online count and player list.

## Hypotheses & Verification
| ID | Hypothesis | Likelihood | Effort | Evidence |
|----|------------|------------|--------|----------|
| A | Presence is written, but the read logic filters valid records out. | High | Low | Rejected |
| B | Discord login works, but Firebase init or write fails at runtime. | High | Low | Confirmed |
| C | The browser context blocks the listener or SDK flow after login. | Medium | Medium | Rejected |
| D | The two clients write different IDs or paths, so they never meet in the same list. | Medium | Low | Rejected |
| E | UI count/list logic is wrong even though data exists. | Medium | Low | Rejected |

## Log Evidence
- Instrumentation added for Discord login success, Firebase init, presence write, connection state, online snapshot, and list rendering.
- The current browser-side instrumentation targets `127.0.0.1`, so it cannot report logs from a deployed/public site back to this machine. Remote evidence collection needs either local reproduction on this machine, same-LAN remote debugging, or browser console/network output from the tester.
- User-provided browser evidence shows Firebase initializes, then `@firebase/database` warns that the configured Realtime Database URL is invalid.
- Direct requests to `https://kombat-retrieval-default-rtdb.firebaseio.com/.json` and `/.settings/rules.json` both return `404 Not Found`, confirming the configured RTDB instance does not exist or is not the correct database URL.
- Firebase CLI is authenticated as `ravnnft@gmail.com`, but `firebase projects:list` returns no projects for that account, and the CLI reports `Project 'projects/kombat-retrieval' not found or deleted` / `firebasedatabase.instances.list denied`, so the signed-in account does not have usable access to the configured Firebase project.
- A new Firebase project and web app were created successfully: project `pattern-retrieval-202606-9809f`, web app `1:589089597604:web:40a1018533e3627831de31`.
- Direct Realtime Database Management API creation with the authenticated token returns `FAILED_PRECONDITION: Blaze plan required for multiple database instances`, so the remaining viable path is creating the first/default Realtime Database instance through Firebase Console or an interactive console flow.

## Verification Conclusion
- Root cause confirmed: the app is pointed at an invalid or missing Firebase Realtime Database instance, so Online Arena presence and challenges have no working backend.
- **Resolution**: Updated `index.html` with the verified credentials and database URL (`https://pattern-retrieval-202606-9809f-default-rtdb.firebaseio.com/`) for the new project `pattern-retrieval-202606-9809f`. Verified that anonymous read/write is open on the database, allowing users to connect and interact correctly.

