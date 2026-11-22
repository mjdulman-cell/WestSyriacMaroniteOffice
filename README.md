# Maronite Divine Office

An installable, offline prayer book for the seven-day Maronite weekly cycle. The application has no runtime dependencies, account, subscription, or third-party requests. It includes 49 office entries drawn from 48 supplied Word documents. Every entry has a source notice because at least one shared prayer or dismissal is referenced without its full text. See [Translation gaps](docs/translation-gaps.md) for the additional omissions and uncertain readings.

## Run on a computer

From the extracted project folder:

```sh
python3 -m http.server 8000 --bind 127.0.0.1 --directory dist
```

Open http://localhost:8000/ and wait for **Ready for offline reading**. The address bar’s current hash can be bookmarked. On supported desktop and Android browsers, use the browser’s Install or Add to Home Screen action.

## Host the app

1. Upload the **contents of `dist/`** to an HTTPS static host. Keep the directory structure, including `prayers/`. The server should serve `index.html` for the app’s directory and return 404 for missing assets. There is no backend or build step for the supplied release.
2. Open the hosted URL and verify that **Ready for offline reading** appears. HTTPS is needed for the service worker outside localhost. A `file://` copy or a plain HTTP LAN address does not support the installation workflow.
3. Disconnect the test device from the network, reopen the app, and try a different day and hour. Reconnect to receive future updates.

The app works at a site root or subdirectory because asset paths and the installation scope are relative. Configure short cache lifetimes or revalidation for `sw.js` and the static files. Publish each update as one release. Once a browser downloads an updated weekly cycle, close all the app’s tabs or windows and open it again to activate that version.

## Install on iPhone or iPad

1. Open the hosted HTTPS address in **Safari**.
2. Use **Share**, then **Add to Home Screen**. If the action is hidden, look under **Edit Actions**. Enable **Open as Web App** if shown, then tap **Add**.
3. Open the new Home Screen icon while still connected. Wait for **Ready for offline reading** in that app instance.
4. Test it in Airplane Mode. After installation, prayer texts, theme, text size, and reading position are stored by the browser on that device. Clearing site data removes those copies and preferences.

[Apple’s iPhone installation instructions](https://support.apple.com/guide/iphone/open-as-web-app-iphea86e5236/ios)

## Add or correct a prayer

- Edit `dist/prayers/<weekday>-<hour>.json` in UTF-8. The ordered `blocks` array holds the actual text. A block’s `text` is displayed verbatim, and `kind` and `section` determine visual treatment. Keep existing block IDs stable so saved positions continue to work. For an added passage, use a new unique ID such as `p301` and insert the block at the intended position in the array. Use `kind: "heading"` with a `level` of 2 or 3 for a table-of-contents entry.
- Reflect any change to `issues` or `passageCount` in both that file and its row in `dist/prayers/index.json`. Keep a record of the replacement source and update the gap notes and audit when making an editorial change. An unresolved replacement should keep its notice.
- Run `python3 tools/build_release.py` after edits. It regenerates `dist/sw.js`, including the offline asset list and a cache revision derived from the release. Upload the refreshed `dist/` together.
- To reimport the original Word edition, install `python-docx`, then run `python3 tools/import_prayers.py /path/to/word-files` followed by the release command. The importer expects the exact filenames listed in the audit and the supplied Sunday split. It overwrites the generated office data and audit. Apply approved changes to the Word sources first if they should survive a reimport. Word originals were supplied separately and are identifiable by SHA-256 in the audit.

## Contents and verification

- `dist/` contains the ready-to-host application and editable prayer data.
- `docs/source-audit.csv` records the complete weekday and office inventory, Word filename, paragraph ranges, counts, and original file hash.
- `docs/translation-gaps.md` describes omissions, source precedence, and weekday assignments.
- `docs/verification.md` records completed tests and platform limits.
- `tools/` contains the Word importer and the service-worker release builder.
