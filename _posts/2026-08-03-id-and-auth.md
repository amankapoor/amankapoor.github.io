---
title: ID & Auth
date: 2026-08-03
published: false
layout: post
---
Identity is person verifying this email/phone etc is their. That is human side.  
Auth is allowing this person to access allowed resources on our server. This is server side.  

**Verification** asks *"does this person control this phone number?"* — one moment, one yes or no, then it forgets you.  
**Authentication** asks *"who is this, across every visit?"* — it keeps a user record, issues a session, and remembers you tomorrow.

**Most Indian "login" vendors only do the first.** MSG91, Truecaller, Twilio, the silent network APIs — they hand back "yes, this number is real" and the rest is yours to build.

**OTPless is the odd one out.** It genuinely tries to do both: sessions, tokens, passkeys, an account model. That's why it was the only India-origin product in the survey I'd call an identity provider at all.

**Yes, they integrate — and that's the normal pattern.** The verification step is the *challenge*; the auth provider is what happens after.

Concretely, for us: someone proves their phone via OTPless or silent network authentication → byld looks up or creates the identity row → byld issues its own token. The verifier is swappable; the identity is ours.

**This is exactly why decoupling identity first was the right call.** We can change how someone proves who they are — SMS today, WhatsApp tomorrow, silent network after that — without any of it touching who they *are* in our database.