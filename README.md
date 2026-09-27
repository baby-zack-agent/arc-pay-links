# ArcPay Links

**Request USDC on Arc — share a link, get paid. No account. No backend.**

A tiny static web app built for the [Arc Microgrants](https://dorahacks.io/hackathon/arc-microgrants)
program (deadline Oct 14, 2026).

## What it does

1. **Generator view** — enter your Arc wallet address, a USDC amount, and an optional
   note. The app builds a shareable payment link like:

   ```
   https://<host>/?to=0xAbc…&amount=25.00&memo=Logo%20design
   ```

2. **Payment view** — anyone who opens that link sees the amount, recipient, and note,
   then taps **Connect wallet & pay**. Their wallet (MetaMask, Rabby, …) switches to
   (or adds) Arc mainnet and sends the USDC via Arc's native USDC ERC-20 contract.
   The app shows the transaction hash with an explorer link on confirmation.

Everything runs in the visitor's browser. There is no server, no database, no API
key, and the app never sees private keys — the user's wallet signs everything.

## Run it locally

```bash
cd arc-pay-links
python3 -m http.server 8000
# open http://localhost:8000
```

Note: it must be served over HTTP(S) — `file://` links are not shareable.

## Host it

Any static host works: GitHub Pages, Cloudflare Pages, Netlify, Vercel, IPFS, …
It's a single `index.html` with one CDN dependency (ethers.js v6, SRI-pinned).

## Arc mainnet details used

| Field | Value |
|---|---|
| Chain ID | `5042` (hex `0x13B2`) |
| RPC | `https://rpc.mainnet.arc.io` |
| Explorer | `https://explorer.arc.io` |
| USDC (ERC-20 interface) | `0x3600000000000000000000000000000000000000` |
| USDC decimals (ERC-20) | `6` |
| Native gas token | USDC (18 decimals at protocol level; same underlying balance) |

**Verified from official sources (Sep 22, 2026):**

- [Building with USDC on Arc: Two Interfaces for One Token](https://www.arc.network/blog/building-with-usdc-on-arc-one-token-two-interfaces)
  — arc.network official blog; confirms USDC as native gas (18 decimals) and the
  ERC-20 interface at `0x3600…0000` with 6 decimals.
- [circlefin/skills — use-arc SKILL.md](https://github.com/circlefin/skills/blob/HEAD/plugins/circle/skills/use-arc/SKILL.md)
  — Circle's official repo; confirms chain ID `5042`, RPC `https://rpc.mainnet.arc.io`,
  explorer `https://explorer.arc.io`, and USDC at the same predeploy address on mainnet
  and testnet.

No contract addresses were guessed. The app hard-codes only the values above.

## Design & safety notes

- **Input validation:** recipient must be a valid EIP-55 address (`ethers.isAddress`);
  amount must be numeric, > 0, ≤ 6 decimals, and within a sane upper bound; memo is
  trimmed and capped at 140 chars.
- **No HTML injection:** all user-supplied values (address, amount, memo) are rendered
  with `textContent`, never `innerHTML`. The one `innerHTML` use (`setStatus`) only
  ever receives internally-built strings (tx hashes, explorer URLs).
- **Self-payment guard:** paying your own link is blocked with an explanatory message.
- **Balance pre-check:** the app reads your USDC balance first and aborts with a clear
  message instead of submitting a doomed transaction.
- **Network handling:** auto-switches to Arc, or prompts `wallet_addEthereumChain`
  (error 4902) with the verified parameters above.
- **Dependency pinning:** ethers.js v6.13.4 from jsDelivr with a Subresource Integrity
  hash computed from the actual file (not copied from docs).
- **Graceful failure:** no wallet → install prompt; CDN failure → visible error;
  malformed link params → explanatory error instead of a blank page.

## Grant-submission checklist (for the operator)

- [ ] Deploy this folder to a public static host → live URL on Arc mainnet (the app
      itself targets Arc mainnet; "deployment" here means the hosted frontend).
- [ ] Push to a public GitHub repo.
- [ ] Submit via the DoraHacks page with the live link + repo + short description.
- [ ] Builder profile: GitHub / X / Farcaster link.

No placeholders remain in the code — all Arc network values are verified above.
