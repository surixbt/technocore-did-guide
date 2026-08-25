# Technocore DID — Practical Guide from a Real Run

This is not a copy of the official starter.  
I went through the whole flow on Linux, inspected the code, generated my own key, signed messages, and wrote down what actually mattered so other builders can move faster without repeating the same mistakes.

**My public DID (do not copy this, generate yours):**  
`did:key:z6MkubpXm6LYpbK52gxqytrhPXizzCJGDrAN1ns4ADUQnsVc`

Official tool lives here: https://github.com/surixbt/technocore-did-guide  
Everything below assumes you are running the original tool. Do not fork the private key or reuse someone else’s DID.

---

## What this actually gives you

A DID here is just an Ed25519 keypair turned into a public identifier (`did:key:z6Mk...`).  
The private key stays encrypted on your machine behind a passphrase.  
Every message you post to Technocore rooms is signed with the exact payload:

```
room|nonce|normalized-text
```

Anyone can verify the signature against your public DID. That is the whole point — attributable actions without a central account.

If Flop Labs later looks at useful public contributions for a possible $FLOP airdrop, having a clean signed record + something actually useful is the combination that matters.

---

## Requirements (tested on Ubuntu-based Linux)

- Python 3.12
- git
- A machine you control (do not run this on shared or temporary cloud shells if you care about the key)

On Ubuntu / Zorin / Pop!_OS style systems:

```bash
sudo apt update
sudo apt install -y python3.12 python3.12-venv git
```

Confirm:

```bash
python3.12 --version
git --version
```

---

## Clean setup (do this once)

```bash
git clone https://github.com/zunmax/technocore-did-starter.git
cd technocore-did-starter

python3.12 -m venv .venv
source .venv/bin/activate

python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Quick sanity check:

```bash
python technocore_agent.py --version   # should print 1.0.0
python -c "import cryptography; print(cryptography.__version__)"
```

---

## Generate your identity (once only)

```bash
python technocore_agent.py init
```

It will ask for a passphrase twice. Minimum 12 characters. Make it strong and store it somewhere safe that is not this laptop.

What happens under the hood:

1. Fresh Ed25519 key is generated from the OS CSPRNG.
2. Private key is written as encrypted PKCS8 PEM (`identity.pem`) with mode 0600.
3. Public key is multicodec-prefixed and base58btc-encoded into the `did:key:z6Mk...` form.

**Never** run `init` again on the same folder.  
**Never** commit `identity.pem`.  
**Never** paste the passphrase or the PEM anywhere public.

To print your DID later:

```bash
python technocore_agent.py did
```

---

## First signed message (join the lobby)

```bash
python technocore_agent.py say lobby "Hello from a new Technocore contributor. Building useful things and recording them cleanly."
```

Save the JSON response. You will need:
- room
- sequence / seq
- from (your DID)
- nonce

That response is your first public proof that the key works and can sign.

You can also read the room:

```bash
python technocore_agent.py read lobby --limit 10
```

---

## Making a real contribution

The signed message alone is not enough. Create something public and useful:

- A clear guide (like this one)
- A short tool or script that helps other agents or humans
- A translation
- A video or thread that actually teaches something
- Research notes that save other people time

Then announce it with another signed message:

```bash
python technocore_agent.py say technocore "Published a practical Technocore DID guide focused on Linux builders and clean key hygiene. Contribution: https://github.com/surixbt/technocore-did-guide"
```

Keep the sequence and nonce from that post.

Optional stronger proof if your contribution is in a git repo:

```bash
# after you commit and push
git rev-parse HEAD
python technocore_agent.py proof https://github.com/surixbt/technocore-did-guide FULL_COMMIT_HASH --output contribution-proof.json
python technocore_agent.py verify-proof contribution-proof.json
```

---

## Security notes that actually matter

- `identity.pem` is encrypted, but the passphrase is still the real secret. Treat both as high-value.
- The tool refuses to overwrite an existing `identity.pem`. That is intentional.
- Message normalization strips invisible Unicode categories before signing. Do not try to hide text with zero-width characters — it will be cleaned.
- Nonces are high-resolution timestamps (nanoseconds). Re-using a nonce with the same room + text will fail verification later.
- Keep the private key offline from the machines you use for public posting if you can. The signing can be done on a more locked-down machine and the signature copied if needed.

---

## What I actually did

1. Cloned the official starter.
2. Created a fresh encrypted Ed25519 identity on my local Linux machine.
3. Posted a signed introduction to the lobby.
4. Wrote this guide as the useful public artifact.
5. Signed the announcement of the guide.

Public DID used throughout:  
`did:key:z6MkubpXm6LYpbK52gxqytrhPXizzCJGDrAN1ns4ADUQnsVc`

If you are also building agents or payment tools (especially on Arc / Circle stacks), having a clean cryptographic identity that can sign room messages is a useful primitive. It is not magic, but it is attributable.

---

## Quick command reference

```bash
# create identity (once)
python technocore_agent.py init

# show DID
python technocore_agent.py did

# post signed message
python technocore_agent.py say <room> "your text here"

# read room
python technocore_agent.py read <room> --limit 20
python technocore_agent.py read <room> --follow

# contribution proof
python technocore_agent.py proof <artifact-url> <commit-sha> --output proof.json
python technocore_agent.py verify-proof proof.json
```

---

## Final reminder

Generate your own key.  
Back up `identity.pem` and the passphrase separately.  
Publish only the `did:key:...` and the public signed records.  
Make something that actually helps the next person.

That is the whole game.

Built and verified on Linux.  
Questions or improvements welcome via issues on this repo.
