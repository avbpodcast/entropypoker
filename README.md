# entropypoker
Entropy Seed Phrase poker method to generate seeds with playing cards


# Entropy Seed Phrase Poker: What Does This Do?

**Entropy Seed Phrase Poker** is a web-based tool designed to generate secure, random seed phrases using a unique card-drawing game powered by high-quality entropy from the drand network. Here's a technical breakdown of how it works, why it’s fair, and how you can use it to create a seed phrase.

---

## Overview of Functionality

This application simulates a poker-like game where users draw pairs of cards—one from a blue deck (32 cards: ranks 1–8 across four suits) and one from an orange deck (64 cards: ranks 1–13 across four suits, plus special "X" cards). Each pair maps to a word from the BIP-39 wordlist (2048 words) via a deterministic formula. By drawing 11 or 23 pairs (user-selectable), you generate a seed phrase with 11 or 24 words (the latter including a checksum). The randomness driving the card shuffles comes exclusively from online entropy fetched from the drand network, ensuring cryptographic-grade unpredictability.

---

## Avoiding Shuffle Bias

To shuffle the decks, the application uses the **Fisher-Yates (or Knuth) shuffle algorithm**, which guarantees an unbiased permutation when paired with a fair random number generator. The `LavaRand` class provides this randomness by fetching entropy from `https://drand.cloudflare.com`, a public randomness beacon operated by the League of Entropy. The `getRandomInt` method employs **rejection sampling** to produce uniform random integers:

- **How it works**: Each byte (0–255) from the entropy buffer is tested against a threshold (`maxSafe`), rejecting values that would bias the distribution. For example, shuffling the orange deck (64 cards) uses `maxSafe = 192` (since `256 / 64 = 4`, and `4 * 64 = 192`). Bytes 192–255 are discarded, ensuring each index 0–63 has an equal chance (192 / 256 = 3 occurrences per index).
- **Why it’s unbiased**: Without rejection, a simple modulo operation (`byte % 64`) would favor lower indices (e.g., 0–63 get 4 chances, 64 gets 0), skewing the shuffle. Rejection sampling eliminates this bias, making every permutation equally likely.

---

## Fairness of Card Pairs

Each card pair is fair because:

- **Deck Independence**: The blue and orange decks are shuffled separately using independent entropy, ensuring no correlation between draws.
- **Mapping to Words**: The pair’s indices (blue: 0–31, orange: 0–63) are combined via `pairIndex = blueIndex * 64 + orangeIndex`, yielding 2048 possible outcomes (32 * 64 = 2048), matching the BIP-39 wordlist size. The final word index is `pairIndex % 2048`, but since `pairIndex` is always 0–2047, this is a direct mapping with no bias.
- **Uniformity**: With 4800 bytes of entropy fetched (150 rounds * 32 bytes), and each shuffle using far fewer (e.g., 63 bytes max for orange deck), the entropy pool vastly exceeds the need, ensuring each pair is statistically independent and uniformly distributed.

---

## Shannon Entropy and Fairness

Shannon entropy quantifies the unpredictability of the seed phrase:

- **Formula**: \( H = - \sum p_i \log_2(p_i) \), where \( p_i \) is the probability of each outcome. For a uniform distribution over 2048 words, \( p_i = 1/2048 \), so:
  - Per word: \( H = \log_2(2048) = 11 \) bits of entropy.
  - For 11 words: \( H = 11 * 11 = 121 \) bits.
  - For 23 words (plus checksum): \( H = 23 * 11 = 253 \) bits, reduced slightly by the checksum (typically 4–8 bits), yielding ~245–249 bits.
- **Proof of Fairness**: The drand entropy is cryptographically secure and publicly verifiable, ensuring \( p_i \) is truly \( 1/2048 \) per draw. The Shannon entropy confirms the seed phrase has sufficient randomness for cryptographic use (e.g., Bitcoin wallets require 128–256 bits), making it resistant to brute-force attacks.

---

## Operational Steps

### 1. Start and Entropy Fetching

- **Click "Start"**: The app initializes by fetching 150 rounds of randomness from `https://drand.cloudflare.com/public/latest` and subsequent rounds (e.g., `/public/12345`). Each round provides 32 bytes (64 hex chars), totaling 4800 bytes.
- **Process**: The `LavaRand` class converts this into a buffer of 8-bit values, displayed as a progress bar ("Loading Entropy: 0% to 100%").
- **Fallback**: If the fetch fails (e.g., no internet), it switches to `crypto.getRandomValues()`, a browser-based cryptographic random number generator, ensuring uninterrupted operation. This is indicated by "Shuffling (Fallback)...".
- **Online Data Only**: The entropy fetch is the *only* online interaction. All subsequent shuffling, pair drawing, and word mapping occur locally in your browser—no additional network requests are made.

### 2. Generating a Seed Phrase

- **Initialize**: After clicking "Start", the blue and orange decks are shuffled using the fetched entropy.
- **Draw Pairs**: Click "Draw Pair" to reveal one card from each deck. Each pair maps to a BIP-39 word, displayed incrementally (e.g., "Words: 1 / 11").
- **Auto-Generate Option**: Click "Auto Generate" to draw all remaining pairs automatically (500ms delay between draws).
- **Completion**: At 11 or 23 draws (toggle 11/23-word mode), the app switches to a "Secure Seed Phrase" box, showing the full phrase (e.g., "abandon ability ...").
- **Checksum (Optional)**: Click "Calculate Checksum" to append a valid BIP-39 checksum word for a 24-word phrase, ensuring compatibility with wallet standards.
- **Copy and Wipe**: Copy the phrase to your clipboard, then click "Wipe Seed Data" to securely erase it from memory.

---

## Why Use This?

This tool combines fun (card game) with security (drand entropy) to produce seed phrases you can trust for cryptocurrency wallets or other cryptographic applications. The rejection sampling, uniform pair mapping, and high Shannon entropy guarantee fairness and unpredictability, while the offline processing after entropy fetch ensures privacy. Try it out—start drawing your secure seed phrase today!
