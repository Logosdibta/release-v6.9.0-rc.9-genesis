
# 0L Genesis Ceremony Guide

**Coordinator**: `sirouk`

> ⚠️ **Important**: Only proceed with asynchronous steps after the coordinator confirms the previous sequential blocking steps as completed.

## Genesis Ceremony Steps

### 1. Cleanup
- Delete any previous forked `release-v6.9.0-rc.0-genesis-2` in your GitHub organization.
- Previous clones and testnets leave data in the `.libra` directory, so you need to clean up:
  ```bash
  rm -Rf ~/libra-framework
  rm -Rf ~/.libra/data && rm -Rf ~/.libra/genesis && rm -Rf ~/.libra/secure-data.json
  ```
- Retrieve Validator Address:
  ```bash
  grep 'account_address' ~/.libra/public-keys.yaml
  ```
- Fetch Static IP
```bash
curl -s ipinfo.io | jq .ip
```
Enter your Validator Address, Operator Name, and Static IP in the Genesis Worksheet.

### 2. Fetch source & verify commit hash
```bash
cd ~
git clone https://github.com/0LNetworkCommunity/libra-framework
cd ~/libra-framework
git fetch --all && git checkout main
git log -n 1 --pretty=format:"%H"
```
> **Note**: Provide git hash in the Genesis Worksheet. Only proceed with the following async steps after the coordinator provides the git hash and your results match.

### 3. Build `libra-framework` packages
```bash
cd ~/libra-framework
cargo build --release -p libra -p libra-genesis-tools -p libra-txs -p diem-db-tool
```
- Confirm with "done" in the Genesis Worksheet.

### 4. Prepare `.libra` directory and add GitHub PAT (use classic with repo privileges)
```bash
mkdir ~/.libra/
nano ~/.libra/github_token.txt
```
- Confirm with "done" in the Genesis Worksheet.

### 5. Pre-Genesis Registration
- Ensure you delete any forked version of `release-v6.9.0-rc.0-genesis-2` in your home org before registering.
```bash
cd ~/libra-framework
./target/release/libra-genesis-tools register  --org-github 0LNetworkCommunity --name-github release-v6.9.0-rc.0-genesis-7
```
- Confirm with "done" in the Genesis Worksheet.

### 6. PR Received
(coordinator)

### 7. PR Merged
(coordinator)

### 8. Build JSON_Legacy
```bash
### Fetch Ancestry Data and Snapshot
mkdir -p ~/libra-recovery
wget https://github.com/0LNetworkCommunity/epoch-archive/raw/main/667.tar.gz -O ~/libra-recovery/667.tar.gz
cd ~/libra-recovery && tar -xvzf 667.tar.gz
wget https://raw.githubusercontent.com/sirouk/ol-data-extraction/v-6.9.x-ready/assets/data.json -O ~/libra-recovery/v5_ancestry.json

### Use v5.2 codebase and snapshot to generate recovery.json for seeding v6.9.x state
sudo rm -Rf ~/libra-legacy-v6
cd ~ && git clone -b v6 https://github.com/0LNetworkCommunity/libra-legacy-v6
cd ~/libra-legacy-v6/ol/genesis-tools
cargo r -p ol-genesis-tools -- --export-json ~/libra-recovery/v5_recovery.json --snapshot-path ~/libra-recovery/667/state_ver* --ancestry-file ~/libra-recovery/v5_ancestry.json
md5sum ~/libra-recovery/v5_recovery.json
```
- Confirm `v5_recovery.json` md5 hash in the Genesis Worksheet.

### 9. All nodes added to `layout.yaml` users key
(coordinator)  
> ⚠️ **Note**: Pre-genesis set closes here. Wait for coordinator.

### 10. Pull from genesis repo and build
```bash
# pull and build genesis
cd ~/libra-framework/tools/genesis
GIT_ORG=0LNetworkCommunity GIT_REPO=release-v6.9.0-rc.0-genesis-7 RECOVERY_FILE=~/libra-recovery/v5_recovery.json make genesis
```
- Confirm with "done" in the Genesis Worksheet.

> ⚠️ **Note**: All set. Wait for the coordinator before the next step.

### 11. Start nodes!
Wait for the coordinator, say a prayer, then start!
```bash
~/libra-framework/target/release/libra node --config-path ~/.libra/validator.yaml
```

---
**End of Ceremony Steps**
