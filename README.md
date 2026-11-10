# HashHound

A Python password and hash auditor. It checks password strength, hashes it with SHA-256, checks if it was pwned with HaveIBeenPwned k-anonymity, and tries to crack your own hash with a wordlist. It saves the full report to `hash_report.json`.

Built to learn real password security: strength scoring, hashing, breach checking without leaking the password, and dictionary attacks.

## What It Does

1. Scores password strength by length, upper, lower, digit, special
2. Hashes the password with SHA-256
3. Checks HaveIBeenPwned with k-anonymity, only sends first 5 chars of the hash
4. Tries to crack your own SHA-256 hash with a wordlist
5. Saves all findings to `hash_report.json`

## Functions

### `check_password_strength(password)`
- Input: `password` - string, e.g. `"MyPass123!"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Uses `re.findall` to find upper, lower, digit, special matches
- Scoring: length 12+ = +2, length 8+ = +1, upper = +1, lower = +1, digit = +1, special = +1
- Max score 6
- If score 6 or more: `severity` OK, `message` Strong password
- If score 3 to 5: `severity` MEDIUM, `message` Medium password
- If score under 3: `severity` HIGH, `message` Weak password
- Naming: `password_length` = int, `upper_matches` = list of matches, `findings` = list of dicts, `finding` = one dict

### `hash_password(password)`
- Input: `password` - string, e.g. `"MyPass123!"`
- Output: `hashed` - string, SHA-256 hex digest
- Encodes password to bytes with `utf-8`
- Hashes with `hashlib.sha256`
- Returns hex digest with `hexdigest`
- Naming: `password_bytes` = bytes, `hash_object` = hash object, `hashed` = string

### `check_pwned(password)`
- Input: `password` - string, e.g. `"MyPass123!"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Hashes password with SHA-1, makes it uppercase
- Splits hash into `prefix` first 5 chars, and `suffix` rest of hash
- Gets `https://api.pwnedpasswords.com/range/{prefix}` with timeout 5
- Uses `requests.RequestException` for errors
- Loops each line in response, splits by `:` to get suffix and count
- If suffix matches: `severity` HIGH, `message` Pwned count times
- If no match: `severity` OK, `message` Not found in breach
- K-anonymity: only the prefix is sent, the full password and full hash never leave your machine

### `crack_hash(target_hash, wordlist_path)`
- Input: `target_hash` - string, SHA-256 hex digest of your own test password
- Input: `wordlist_path` - string, e.g. `"common_passwords.txt"`
- Output: list of dicts, each finding has `check`, `severity`, `message`
- Opens wordlist, if missing returns error finding
- Loops each word, strips newline, hashes word with SHA-256
- If hashed word equals target hash: `severity` HIGH, `message` Cracked password is word, returns
- After loop, if no match: `severity` OK, `message` Not cracked with wordlist
- The "Not cracked" block must be outside the loop, or it will only check the first word

### `main()`
- Takes password from `sys.argv`
- Calls `check_password_strength`, `hash_password`, `check_pwned`, `crack_hash`
- Combines lists into `combined_findings`
- Saves to `hash_report.json` with `json.dump` and `indent=4`
- Prints each finding as `[check] Severity: severity - message`

## Quick Start

```bash
git clone https://github.com/YOUR-USERNAME/HashHound.git
cd HashHound
pip install -r requirements.txt
