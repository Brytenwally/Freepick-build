# Freepick-build

Builds the Windows server (worldserver and authserver) for the Freepick game mode. It combines:
- [azerothcore-wotlk-coa](https://github.com/Brytenwally/azerothcore-wotlk-coa) on branch `main`: the Area 52 server
  source with the upstream CoA updates applied, and the CoA modules it already contains;
- [mod-playerbots](https://github.com/Brytenwally/mod-playerbots) on branch `area52-freepick`: the CoA bots with the
  dungeon and raid strategies of `coa-bot-strategies`, plus the Freepick Hero work (Legendary classes, their
  rotations and triggers).

No other modules are added: no 10 Man World and no ALE.

## Running a build

1. Open **Actions**, pick **Windows build**, and click **Run workflow**. You can change the branches in that dialog.
2. The build takes about one to two hours. Click the run to follow each step.
3. When it is green, download **freepick-server-windows** under **Artifacts** at the bottom of the run page.
4. Unzip it over your server folder. Keep your own `.conf` files and the `Data` folder.
   - `versions.txt` in the zip says which commit of each repository went into the build.

The zip also holds the OpenSSL DLLs (`libcrypto-3-x64.dll`, `libssl-3-x64.dll`) the server was built with; keep them
next to the exe files.

The databases and the `Data` folder (dbc, maps, vmaps, mmaps) are not part of the build.

The worldserver applies the core's pending database updates the first time it starts. Back up the auth, characters and
world databases before starting a new build, and try a build with many new updates on a copy of them first.

## Identical function folding

The build links with `/OPT:NOICF`. MSVC's Release default (`/OPT:ICF`) merges functions with identical bodies, and
the core tells implemented aura and spell effect handlers from unimplemented ones by comparing function pointers:
the empty `AuraEffect::HandleNoImmediateEffect` became the same function as `HandleNULL`. Free Pick then rejected
every Mystic Enchant built on such an aura (spell modifiers, proc triggers, periodic damage and healing): 84 of 3,822
connected, against about 3,200 on a Linux build.

## MySQL version

The worldserver refuses to start unless its `libmysql.dll` is the exact MySQL version it was compiled against. The
error is "Used MySQL library version ... does not match" (ACE00046).

The **mysql_version** field of the Run workflow dialog picks that version. Its default is `8.4.2`, the version of
the portable server. The build downloads that version, compiles against it and puts its `libmysql.dll` in the zip.

Use the zip's `libmysql.dll`, not an older one.
