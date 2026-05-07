# JWT Explained — A Beginner's Guide

## What is JWT?

**JWT = JSON Web Token** (pronounced "jot")

Think of it like a **digital ID card** that proves who you are.

## Real-World Analogy

Imagine going to a music festival:

```
1. You go to the ticket counter (GitHub)
2. You show your ID and buy a ticket
3. They give you a WRISTBAND (this is the JWT)
4. The wristband has info printed on it:
   - Your name
   - Which areas you can access (VIP, backstage)
   - When it expires (end of the day)
5. You show the wristband to enter any stage (LaunchDarkly)
6. The security guard READS the wristband — doesn't need to call the ticket counter
7. At the end of the day, the wristband is CUT (token revoked)
```

## What's Inside a JWT?

A JWT has 3 parts separated by dots:

```
eyJhbGciOiJSUz.eyJzdWIiOiJ1c2Vy.SflKxwRJSM
|___ HEADER ___|.___ PAYLOAD ____|._SIGNATURE_|
```

| Part | What It Contains | Analogy |
|---|---|---|
| **Header** | How it was signed (algorithm) | The type of wristband material |
| **Payload** | The actual data (who you are, what you can do, when it expires) | The info printed on the wristband |
| **Signature** | Proof it's genuine (signed by the issuer) | The hologram sticker that proves it's not fake |

## The Header

Tells the receiver how the token was signed:

```json
{
  "alg": "RS256",    ← the signing algorithm (RSA + SHA-256)
  "typ": "JWT"       ← the token type
}
```

This is like saying "this wristband was sealed with hologram type X."

## The Payload (the important part)

Contains "claims" — pieces of information about the user and the token:

```json
{
  "sub": "user_id:213455",               ← WHO you are
  "preferred_username": "arifshaikh",     ← your GitHub username
  "user_id": "213455",                    ← your GitHub user ID
  "iss": "https://github.com",           ← WHO issued this token (GitHub)
  "aud": "https://mcp.launchdarkly.com", ← WHO it's meant for (LaunchDarkly)
  "exp": 1714000300,                      ← WHEN it expires (5 min from now)
  "iat": 1714000000                       ← WHEN it was issued
}
```

### Common Claims Explained

| Claim | Full Name | Meaning | Example |
|---|---|---|---|
| `sub` | Subject | Who this token is about | `user_id:213455` |
| `iss` | Issuer | Who created and signed this token | `https://github.com` |
| `aud` | Audience | Who this token is intended for | `https://mcp.launchdarkly.com` |
| `exp` | Expiration | When this token stops being valid | `1714000300` (Unix timestamp) |
| `iat` | Issued At | When this token was created | `1714000000` (Unix timestamp) |

## The Signature

The signature proves the token hasn't been tampered with:

```
SIGNATURE = RSA_SHA256(
  base64(header) + "." + base64(payload),
  GitHub's_private_key
)
```

- **GitHub signs it** with its private key (only GitHub has this)
- **LaunchDarkly verifies it** with GitHub's public key (anyone can check it)
- If anyone changes even one character in the header or payload, the signature won't match — the token is rejected

This is like the hologram on your wristband — you can look at it to verify it's real, but you can't create a fake one.

## How JWT Works in This Plugin

```
Step 1: You @-mention the agent in a PR
            |
            v
Step 2: GitHub says "I know this user, let me create a JWT"
        GitHub SIGNS the JWT with its private key
        (like stamping a wristband with an official seal)
            |
            v
Step 3: GitHub sends the JWT to LaunchDarkly:
        "Hey, here's proof this person is who they say they are"
            |
            v
Step 4: LaunchDarkly VERIFIES the JWT:
        - Checks GitHub's signature (is the seal real?)
        - Checks expiry (is it still valid?)
        - Checks the username (who is this person?)
            |
            v
Step 5: LaunchDarkly says "OK, I trust GitHub. Here's an access token"
        (like trading your wristband for a backstage pass)
            |
            v
Step 6: The agent uses that access token to create/list flags
            |
            v
Step 7: Job done -> token is REVOKED (wristband is cut)
```

## Why JWT Instead of Just a Password or API Key?

| Password / API Key | JWT |
|---|---|
| Stored somewhere (can be stolen) | Created on the fly, never stored |
| Lives forever (until you change it) | Expires in 5 minutes |
| Same key used every time | Fresh token every time |
| If leaked, attacker has permanent access | If leaked, it expires in minutes |
| You have to manage and rotate it | GitHub handles everything automatically |

## The Key Insight

**Nobody stores a secret.** GitHub creates a fresh, signed JWT every time the agent runs. LaunchDarkly trusts it because it can verify GitHub's signature. After 5 minutes, the JWT is useless. That's why it's more secure than storing an API key.

## Another Analogy: Boarding Pass

Think of JWT like a **boarding pass** at an airport:

| Airport | JWT in This Plugin |
|---|---|
| Airline issues a boarding pass | GitHub signs a JWT |
| Has your name, flight, seat | Has your username, permissions, expiry |
| Barcode can be scanned to verify | Signature can be verified with public key |
| Only valid for one flight | Only valid for 5 minutes |
| Don't need to show passport at every gate | Don't need to re-authenticate at every API call |
| Worthless after the flight | Revoked after the agent job completes |

## How Verification Works (Public/Private Keys)

```
GITHUB (has the private key):
  "I'll sign this JWT so everyone knows it came from me"
  JWT = sign(payload, PRIVATE_KEY)

       The JWT travels over the internet...

LAUNCHDARKLY (has the public key):
  "Let me check if GitHub really signed this"
  valid = verify(JWT, PUBLIC_KEY)

  IF valid:
    "Yes, GitHub signed it. I'll trust it."
  ELSE:
    "This is fake or tampered with. Rejected."
```

The **private key** is like a royal seal — only the king (GitHub) has it.
The **public key** is like knowing what the royal seal looks like — anyone can check if a letter has the real seal.

## JWT vs Other Token Types

| Type | How It Works | Used Where |
|---|---|---|
| **API Key** | A static string you store and send with every request | `LD_API_KEY` (fallback in this plugin) |
| **Session Cookie** | Server stores your session, gives you an ID | Web apps after login |
| **OAuth Access Token** | Opaque string, server must look it up to validate | After OAuth flow |
| **JWT** | Self-contained, signed, can be verified without calling the issuer | OIDC in this plugin (primary) |

The key difference: a JWT is **self-contained**. LaunchDarkly can verify it by checking the signature — it doesn't need to call GitHub and ask "is this token real?" That makes it fast and decoupled.

## Summary

1. **JWT is a signed, self-contained token** with info about who you are and when it expires
2. **GitHub creates it** and signs it with a private key
3. **LaunchDarkly verifies it** using GitHub's public key
4. **It expires in 5 minutes** and is revoked after use
5. **No secrets are stored** — a fresh JWT is created every time
6. **It's more secure than API keys** because it's short-lived and auto-managed
