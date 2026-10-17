REBUILD INSTRUCTIONS — NEEDPEDIA PROJECT SEED
=============================================

This folder holds step-by-step instructions for rebuilding each Needpedia
system from the seed. It replaces the "rebuild/ not written yet" note in
START-HERE.txt and MISSING.txt.

Rule of thumb: every file that used to contain a password, key or token now
says REPLACE_ME. Search for it before starting a system:

    grep -rn "REPLACE_ME" code/ knowledge-bases/ documents/

Make your own values. Never reuse Needpedia's originals — they were removed
on purpose.

READ FIRST (once)
-----------------
1. START-HERE.txt — what this seed is and the suggested order of systems.
2. inventory.txt — every system, what it runs on, the accounts it needs,
   and where its rebuild instructions live.
3. documents/developer-wiki/Needpedia-Test-Site-Package.md — the one
   already-tested guide: running the site itself on your own computer.

WHAT EXISTS NOW
---------------
- [x] Test site package (the main Needpedia site on your own computer):
      documents/developer-wiki/Needpedia-Test-Site-Package.md
      and code/needpedia-test-sites/
- [x] The Nexus document server: NEXUS.md in this folder (written for
      this edition of the seed).
- [ ] Main site on a rented server: follow the test-site package, then
      see inventory.txt section 1 for the accounts you must make
      (server, domain, email sending, Mapbox, OpenRouter).
- [ ] Florence, Lotte, Adele and the public email page: inventory.txt
      sections for each; their code is in code/florence-greeter,
      code/lotte-assistant and code/adele-vc-email; their knowledge bases
      are in knowledge-bases/ (replace Needpedia's contacts with yours).
- [ ] Phone notification app: code/phone-app (replace the Firebase keys,
      see MISSING.txt).

WHAT IS NOT BUILT (anywhere in the seed)
----------------------------------------
- Backups: no backup system exists. Databases, the Nexus content folder
  and the assistants' state are not backed up by any of this code. Set up
  your own (host snapshots plus `pg_dump` on a schedule is enough to
  start) and write the instructions back into this folder.
- The server-to-server network: following people on other fediverse
  servers and showing their posts works; servers following each other
  and automatic backups between them are a plan only. See the bottom of
  MISSING.txt ("THE NETWORK OF NEEDPEDIA SERVERS").

HOW TO ADD ANOTHER REBUILD FILE
-------------------------------
One file per system, same shape as NEXUS.md: what it is, what you need,
numbered steps, how to tell it worked, known limits. Tick its line in the
checklist above when you add it.
