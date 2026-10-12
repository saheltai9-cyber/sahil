# CivicFix — Complete Code aur PC Setup

Ye tumhari CivicFix website ka original application source hai:
https://civiclens-ai.sahiltai005.chatgpt.site

Is export mein PC par chalane ke liye local server bhi diya hai. Original citizen/admin UI aur production API code isi project mein hain.

## 1. PC par kaise chalao

1. ZIP extract karo. `civicfix-complete-code` folder VS Code mein kholo.
2. Node.js **24.x** install karo. Is export ko Node.js 24.19.0 par check kiya gaya hai.
3. VS Code mein **Terminal → New Terminal** kholo. Terminal isi folder mein hona chahiye jahan `package.json` hai.
4. Ye commands ek-ek karke chalao:

```bash
npm ci
npm run dev
```

5. Browser mein **http://localhost:3000** kholo.

| Page | Local address |
| --- | --- |
| Public community dashboard | http://localhost:3000 |
| Citizen panel | http://localhost:3000/app |
| Admin panel | http://localhost:3000/admin |
| Login | http://localhost:3000/login |
| Sign up | http://localhost:3000/signup |

`npm run dev` pehle frontend/backend build karta hai, phir server start karta hai. Code edit karne ke baad terminal mein **Ctrl+C** dabao aur command dobara chalao. HTML par double-click karne se complete app nahi chalegi, kyunki ismein backend bhi hai.

## 2. Admin login set karna

1. `.env.local.example` ki copy banao aur uska naam `.env` rakho. VS Code Explorer mein hidden-name files bhi dikhti hain.
2. `.env` mein apni email likho:

```dotenv
APP_ORIGIN=http://localhost:3000
PORT=3000
ADMIN_EMAILS=your-email@example.com
```

3. `your-email@example.com` ko apni actual email se replace karo. Same email ka Firebase account use karo. Zarurat ho to Sign up page se account banao aur verification email ka link kholo.
4. Server restart karo. Ab same account se `/admin` par login karo.

Admin password tumhare Firebase account ka password hai. Koi default admin password ya hardcoded username nahi hai. `.env` local development ke admin access ko configure karta hai; hosted website ke settings isse automatically change nahi hote.

## 3. Firebase login

Original project mein Firebase email/password aur Google sign-in ka code hai. Firebase console settings aur internet access login ke liye zaroori hain; web config hone se provider settings ka access nahi milta.

- Firebase Authentication mein **Email/Password** provider enabled hona chahiye.
- Google sign-in use karna ho to **Google** provider enable karo aur Authentication → Settings → Authorized domains mein `localhost` add/check karo.
- Agar login fail ho, error message dekho; frontend/backend tests real Firebase account login verify nahi karte.
- Password aur account creation Firebase handle karta hai. Local complaints SQLite mein aur photos local folder mein save hoti hain.

Apna alag Firebase project use karna ho to ye teen jagah consistent update karo:

1. `public/firebase-auth.js`: apne project ka Firebase web config.
2. `server/firebase-auth.js`: `project` mein same Firebase project ID.
3. `server/worker.js`: Content Security Policy ke Firebase auth-domain URLs.

Phir `npm run dev` dobara chalao.

## 4. Code mein kya hai

| File/folder | Kaam |
| --- | --- |
| `public/index.html`, `public/app.css` | Page shell, citizen/admin UI aur responsive styles |
| `public/app.js` | Dashboard, complaint forms, themes, charts, CSV/PDF exports aur controls |
| `public/map.js` | Interactive map, pins, clustering, filters aur location selection |
| `public/firebase-auth.js` | Email/password login, Google sign-in, email verification aur password reset |
| `server/worker.js` | Complaint APIs, uploads, notifications, access management aur admin permissions |
| `server/firebase-auth.js` | Firebase token verification aur server-side role checks |
| `db/schema.ts`, `drizzle/` | Database schema aur SQL migrations |
| `scripts/build.mjs` | Frontend/backend bundling |
| `scripts/local-server.mjs` | Local SQLite/photo storage adapter; same original Worker/API chalata hai |
| `tests/` | Existing backend, UI, map aur PDF checks |
| `dist/server/index.js` | Built Worker; source changes ke baad dobara build karo |

Citizen complaint submit kar sakta hai, photo/location add kar sakta hai, progress aur notifications dekh sakta hai. Admin verification, status/department/staff assignment, user access aur analytics manage kar sakta hai. Permissions backend par bhi check hoti hain.

Civic Guide aur insights rules-based hain. Is code ko chalane ke liye OpenAI API key required nahi hai. Government department ko automatic forwarding aur automatic image recognition connected nahi hain.

## 5. Data aur hosting

- Local reports/photos `.civicfix-local/` mein persist hoti hain. Server restart karne par data rahega.
- Purani live website ki complaints, photos, runtime secrets aur passwords ZIP mein nahi hain. Local database fresh start karta hai; reports create karne par data dikhega.
- Local server sirf apne PC ke loopback address par development ke liye chalta hai. Production mein original Cloudflare Worker ko D1 binding `DB`, R2 binding `BUCKET`, `APP_ORIGIN` aur server-side `ADMIN_EMAILS` settings chahiye.
- `.openai/hosting.json` original Site identity/bindings preserve karta hai. Ye export khud se koi deployment nahi karta.

## 6. Build aur tests

```bash
npm run build
npm test
```

Build ke baad tests chalao. Export mein PDF test ka old brand assertion current `Civic Fix` naam se update kiya hai; PDF creation, complaint content aur multiple-page checks retained hain. Local runner, start commands, local env example aur ye guide export ke additions hain.

Node.js SQLite warning dikhe to terminal mein actual error bhi check karo; tested Node.js 24.19.0 runtime mein SQLite warning ke saath bhi build/server/tests run hote hain.

### Common problems

| Problem | Kya check karo |
| --- | --- |
| `npm` command nahi mil raha | Node.js install karke VS Code restart karo |
| `node:sqlite` unavailable | Node.js 24.x use karo |
| Admin access nahi mil raha | `.env` ki email, verified Firebase account aur server restart |
| Google login unauthorized domain | Firebase Authorized domains mein `localhost` |
| Request origin error | Browser URL aur `APP_ORIGIN` exactly same rakho |
| Port 3000 already in use | `.env` mein `PORT=3001` aur `APP_ORIGIN=http://localhost:3001`, phir restart |

Technical architecture ke detailed notes `README.md` mein hain.

Official references: [Firebase Google sign-in](https://firebase.google.com/docs/auth/web/google-signin), [Firebase authorized domains](https://firebase.google.com/docs/auth/web/email-link-auth), [Node environment-file flags](https://nodejs.org/api/cli.html).

Validated: build successful, all 29 existing tests passed, and local HTTP pages/API/auth boundaries plus SQLite persistence after restart checked. Real Firebase sign-in aur browser visual verification is export check mein run nahi hui.

Published source: version 18, commit `f312136ba267b67823317a1d0605b0a66f8db563`. Export prepared: 12 October 2026.
