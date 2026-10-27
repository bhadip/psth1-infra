## Dashboard Status is DISCONNECTED
**Symptom:** Dashboard status is DISCONNECTED.
**Cause:** EA doesn't allow web request list by default for security purpose.
**Fix:** Make sure `https://aegis.prasanti.com` is added in **Tools** > **Options** > **Expert Advisors** > **Allow WebRequest for listed URL**.

## KasmVNC Clipboard Crash ("Kasm encountered an error")
**Symptom:** Browser shows `InvalidStateError: Failed to execute 'getType' on 'ClipboardItem'` and the VNC session freezes or crashes when switching MT4 windows or pasting.
**Fix:**
1. Hard refresh the browser (`Ctrl+Shift+R`).
2. Open the Kasm sidebar menu (left edge of screen) -> **Settings**.
3. Turn **OFF**: `Clipboard Seamless`, `Clipboard Up`, and `Clipboard Down`.

## Second MT4 Instance Not Appearing or Sharing Login
**Symptom:** Opening a second MT4 window just shows the same account/login as the first one.
**Cause:** Clicking the same desktop shortcut twice opens multiple windows of the *same* terminal process.
**Fix:** Ensure you are launching the distinct duplicated folder.
1. Verify two processes are running: `docker exec mt4-desktop ps aux | grep terminal.exe`
2. If only one is running, manually launch the second: `docker exec -d -u abc mt4-desktop /usr/lib/wine/wine64 "/config/.wine/drive_c/Program Files (x86)/MetaTrader 4 - CT/terminal.exe"`
3. Ensure you have a separate desktop shortcut for the CT account.

## Permission Denied when SCP'ing .mq4 files
**Symptom:** `scp: dest open ".../MQL4/Experts/file.mq4": Permission denied`
**Cause:** The bind-mounted `.wine` directory was created by the container as `root` or `abc`, and the host user `tom` lacks write access.
**Fix:** Run this on the `psth1` host:
```bash
ssh -t tom@100.75.220.84 "sudo chown -R 1000:1000 '/home/tom/docker/mt4-server/config/.wine/drive_c/Program Files (x86)/MetaTrader 4/MQL4/'"

## MT4 "can't open program file" Error
**Symptom:** MetaEditor log shows can't open "C:\Program Files (x86)\MetaTrader 4\MQL4\Experts\file.mq4" program file.
**Cause:** Wine internal file mapping permissions or path spacing issues.
**Fix:**
Fix container ownership: docker exec -u root mt4-desktop chown -R abc:abc '/config/.wine/drive_c/Program Files (x86)/MetaTrader 4/MQL4/'
If it persists, use MetaEditor's File -> Open (Ctrl+O) instead of double-clicking in the Navigator.

## Docker Compose YAML Parse Error
**Symptom:** go-yaml load error in parser... did not find expected key when running docker compose up.
**Cause:** Incorrect indentation or invalid characters added to docker-compose.yml (often from automated sed commands).
**Fix:** Open docker-compose.yml in nano or vim, check the indentation of the environment: block, and ensure no stray characters exist. Run docker compose config to validate.
