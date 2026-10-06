# 1v1LB setup

## Files
| File | Commit to git? | What it is |
|---|---|---|
| `index.html` | yes | The site. Has no keys in it. |
| `.env` | **no** (in .gitignore) | Your private settings. |
| `.env.example` | yes | Blank copy of `.env` for other people. |
| `build-config.js` | yes | Turns `.env` into `config.js`. |
| `config.js` | **no** (in .gitignore) | Generated. The site loads it. |
| `firestore.rules` | yes | Database rules. |
| `firebase.json` | yes | Hosting settings. Never uploads `.env`. |

## Run the site
1. `node build-config.js` (needs Node.js; it writes `config.js` from `.env`)
2. Upload `index.html` and `config.js` to your host. **Never upload `.env`.**
   - Firebase Hosting: `firebase deploy --only hosting` (firebase.json already skips `.env`).
   - Netlify/Vercel: set the same names as environment variables in their dashboard and use `node build-config.js` as the build command.

Change a setting in `.env`? Run `node build-config.js` again and re-upload `config.js`.

> The Firebase values are meant to be public. Any browser that opens the site can see them, so `.env`
> only keeps them out of your git repo. What protects your data is `firestore.rules`. You can also lock
> the API key to your site address in Google Cloud console > APIs & Services > Credentials > your
> browser key > "Websites".

## Put it on GitHub Pages (keys stay out of the repo)
1. Upload everything in the GitHub zip to your repo (it has no `.env` or `config.js` in it).
2. Repo **Settings > Secrets and variables > Actions > New repository secret**. Add one secret for each line in your `.env`, with the same names and values:
   `FIREBASE_API_KEY`, `FIREBASE_AUTH_DOMAIN`, `FIREBASE_PROJECT_ID`, `FIREBASE_STORAGE_BUCKET`, `FIREBASE_MESSAGING_SENDER_ID`, `FIREBASE_APP_ID`
3. Repo **Settings > Pages > Source: GitHub Actions**.
4. Push (or run it from the **Actions** tab > Deploy site > Run workflow). The workflow builds `config.js` from the secrets and publishes `index.html` + `config.js`.
5. Firebase console > Authentication > Settings > Authorized domains: add `YOURNAME.github.io` (needed for Google sign-in).
6. Firebase console > Firestore Database > Rules: publish `firestore.rules`. Authentication > Sign-in method: turn on Email/Password.

GitHub's drag-and-drop uploader ignores `.gitignore`, so never drag `.env` or `config.js` into it.

## Sign-in methods
- **Username + password** (no email). Turn on **Email/Password** in Firebase console > Authentication > Sign-in method.
  Behind the scenes a username becomes a private address like `kai@users.1v1lb.invalid` (the `.invalid` domain can never receive mail). There is no password reset for these accounts.
- **Google** (optional). Firebase console > Authentication > Sign-in method > Google > Enable, then Settings > Authorized domains > add your site's domain.

## Where disputes go
Open the Firebase console > Firestore Database > Data > **`reports`**. Each document has a `type`:
`Direct Dispute` (has a proof link), `Both Players Claim The Win`, `Both Players Claim The Loss`,
`Suspicious Pairing - Repeat Opponents`. Players can file reports but nobody can read them from the site; only you can in the console.
Disputes don't change any Elo on their own. To settle one, edit the two players' documents in `users` (`elo`, `wins`, `losses`).

## Rules
`firebase deploy --only firestore:rules` publishes `firestore.rules` for you, no pasting.
