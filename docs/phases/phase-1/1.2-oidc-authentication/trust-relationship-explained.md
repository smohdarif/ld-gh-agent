# Trust Relationship Explained — Why LaunchDarkly Trusts GitHub

## The Trust Problem

```
LaunchDarkly receives a JWT that says:
  "Hi, I'm arifshaikh, GitHub sent me"

LaunchDarkly thinks:
  "Why should I believe you? Anyone could send me a JWT claiming to be from GitHub."
```

For this to work, two things must be true:
1. LaunchDarkly can **verify** the JWT is genuinely from GitHub (not forged)
2. LaunchDarkly has **agreed in advance** to accept JWTs from GitHub

## How Trust Is Established

### 1. GitHub's Public Keys (Automatic, No Setup Needed)

GitHub publishes its public keys at a well-known URL that anyone can access:

```
https://github.com/.well-known/jwks.json
```

This is like GitHub putting its **official seal pattern** in a public display case. Anyone can look at it to verify if a seal is real.

- GitHub **signs** JWTs with its **private key** (kept secret, only GitHub has it)
- LaunchDarkly **verifies** JWTs using GitHub's **public key** (publicly available)
- If the signature matches → the JWT genuinely came from GitHub
- If it doesn't match → it's fake, rejected

This part requires **no prior agreement**. Anyone can verify a GitHub-signed JWT. But verification alone isn't enough — LaunchDarkly also needs to decide whether to **accept** it.

### 2. The Trust Configuration (Manual, One-Time Setup)

An admin must configure LaunchDarkly to trust GitHub as an identity provider. This answers:

- "Should I **accept** JWTs from GitHub?"
- "Which GitHub orgs/users am I willing to serve?"
- "What permissions should I give them?"

```
BEFORE any JWT is ever issued:

+------------------+                    +------------------+
|    GITHUB         |                    |  LAUNCHDARKLY     |
|                  |                    |                  |
| "I can sign JWTs |   Admin sets up    | "I'll trust JWTs  |
|  for my users"   | <----------------> |  signed by GitHub  |
|                  |  trust relationship |  for org X"       |
+------------------+                    +------------------+

This happens ONCE, done by an admin in the LaunchDarkly dashboard.
```

## Real-World Analogy: Hotel and Corporate Badge

```
BEFORE your trip:
  Your company (GitHub) signs a contract with the hotel (LaunchDarkly):
  "Our employees will visit. Here's what our company badges look like.
   Please give them access to conference rooms."

  Hotel registers: "OK, I'll accept badges from this company."

DURING your trip:
  1. Your company gives you a badge (JWT) with your name and department
  2. You walk into the hotel and show your badge
  3. Hotel security checks:
     a. Is this badge from a company we trust? -> YES (GitHub is trusted)
     b. Is the badge real? -> YES (signature matches GitHub's public key)
     c. Is it expired? -> NO (still within 5 minutes)
  4. Hotel gives you a room key (access token)

WITHOUT the contract:
  You show your badge -> Hotel says "I don't know your company. Go away."
```

## The Full Timeline

### Phase 1: One-Time Setup (done by an admin, once)

```
Step 1: Admin installs the GitHub Copilot Extension
        (or sets up the LaunchDarkly integration)

Step 2: LaunchDarkly registers GitHub as a trusted identity provider:
        - "I will accept JWTs from issuer: https://github.com"
        - "I will verify signatures using keys from:
           https://github.com/.well-known/jwks.json"
        - "I will map GitHub users to LaunchDarkly accounts"

Step 3: (Optional) User account linking:
        - GitHub user "arifshaikh" is linked to LaunchDarkly user "arifshaikh"
        - This determines what permissions the access token will have
```

This only happens **once**. After this, the automated flow works every time.

### Phase 2: Every Time the Agent Runs (automated)

```
Step 1: Developer @-mentions agent in PR

Step 2: GitHub signs a JWT:
        "This is arifshaikh from org my-org, working on repo X"

Step 3: GitHub sends JWT to LaunchDarkly's token endpoint

Step 4: LaunchDarkly checks:
        a. "Is GitHub a trusted issuer?"
           -> YES (configured in Phase 1)
        b. "Is this JWT really from GitHub?"
           -> verify signature with public key -> YES
        c. "Is it expired?"
           -> NO (within 5 minutes)
        d. "Do I know this user?"
           -> YES (linked in Phase 1)
        e. "What can this user do?"
           -> look up their LaunchDarkly permissions

Step 5: LaunchDarkly returns an access token scoped to user's permissions

Step 6: Agent creates/lists flags using the access token

Step 7: Token revoked when done
```

## Do GitHub and LaunchDarkly "Talk" Before Issuing a JWT?

**Not directly in real-time.** Here's the key distinction:

| Question | Answer |
|---|---|
| Do they talk to set up trust? | **Yes, but only once** — an admin configures the trust relationship in LaunchDarkly |
| Do they talk every time a JWT is issued? | **No** — GitHub signs the JWT on its own, LaunchDarkly verifies it on its own |
| How does LaunchDarkly verify without calling GitHub? | It uses GitHub's **public keys** (published at `/.well-known/jwks.json`) to check the signature offline |
| What if there's no trust setup? | LaunchDarkly rejects the JWT — "I don't trust this issuer" |

## Why This Is Clever

After the one-time setup, **no real-time communication is needed** between GitHub and LaunchDarkly to verify a JWT:

```
Traditional approach:
  Agent -> LaunchDarkly -> calls GitHub: "Is this token real?" -> GitHub: "Yes"
  (slow, requires GitHub to be available, adds a network call)

JWT approach:
  Agent -> LaunchDarkly -> checks signature using cached public key -> "Yes, it's real"
  (fast, no network call to GitHub, works even if GitHub is briefly down)
```

This is called **decoupled verification** — the verifier (LaunchDarkly) doesn't need to contact the issuer (GitHub) every time. The public key is all it needs.

## What Happens If Trust Is NOT Set Up?

```
Developer: @launchdarkly-agent create a flag

GitHub: *signs a JWT and sends it to LaunchDarkly*

LaunchDarkly:
  1. Receives JWT
  2. Checks issuer: "https://github.com"
  3. Looks up trusted issuers list: GitHub is NOT on the list
  4. REJECTS the JWT
  5. Returns: "401 Unauthorized"

Agent: "Authentication failed. Please configure LaunchDarkly OIDC integration."
```

## What Happens If Account Linking Is NOT Done?

```
Developer: @launchdarkly-agent create a flag

GitHub: *signs JWT with sub: user_id:213455, preferred_username: arifshaikh*

LaunchDarkly:
  1. Receives JWT
  2. Verifies signature -> valid (GitHub is trusted)
  3. Looks up user: "arifshaikh" -> NOT FOUND in LaunchDarkly
  4. Two possible outcomes:
     a. Returns error: "User not linked. Please complete account linking."
     b. Creates a limited-access token (depends on LaunchDarkly's config)
```

This is why the **account linking step** (Phase 1, Step 3) matters — it connects your GitHub identity to your LaunchDarkly permissions.

## Summary

1. **Trust must be set up once** — an admin configures LaunchDarkly to accept JWTs from GitHub
2. **GitHub publishes its public keys** — LaunchDarkly uses these to verify JWT signatures
3. **No real-time communication needed** — after setup, verification is offline using public keys
4. **Account linking maps identities** — connects your GitHub user to your LaunchDarkly permissions
5. **Without trust setup** → JWTs are rejected
6. **Without account linking** → JWTs are valid but LaunchDarkly doesn't know who you are
