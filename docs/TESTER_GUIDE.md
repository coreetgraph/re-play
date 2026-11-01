# re-play: tester guide

Thank you for trying re-play, a player and review tool for VFX and animation shots. This is a **test version**: it may contain
bugs. **Your media is never modified** and **nothing is sent automatically**. Please read `TEST_AGREEMENT.md`.

## 1. Install

- **Windows** (10 / 11, 64-bit): run `re-play_<version>_Windows_<date>.exe`. Its first screen asks for the language
  (French / English); it installs for you only, without administrator rights. Windows may show "Windows protected your PC"
  (the program is not signed): click *More info*, then *Run anyway*. On some PCs (Smart App Control, company machines), an
  unsigned program is blocked with no button to override it: let us know.
- **Mac** (Apple chip M1 or later, macOS 14 Sonoma or later): open `re-play_<version>_Mac_<date>.dmg`, drag `re-play` into
  `Applications`. First launch: double-click `re-play`; macOS says it cannot verify the app (it is not signed by Apple):
  click *Done*. Then open *System Settings > Privacy & Security*, scroll to "re-play was blocked" and click *Open Anyway*
  (your password is asked). After that, nothing more. On a Mac, **Cmd** replaces **Ctrl** everywhere in this guide.
- **Linux** (64-bit, Ubuntu 26.04 or newer; H.264 export needs an NVIDIA card): make `re-play-<version>-x86_64.AppImage` executable (right-click >
  Properties > "Allow executing", or `chmod +x re-play-*.AppImage`), then double-click it.
- **Language**: re-play follows the computer's language (on Windows, the one chosen in the installer). To change it:
  *Preferences > Appearance > Language*, then restart re-play.
- Your version: *Help > About re-play* (on a Mac: *re-play > About re-play* menu) gives the number and date. Quote them in your feedback.
- A test version expires after 60 days (the date is in About; a message warns you two weeks before). Then ask for the new one.
- Check your download if you like: `Get-FileHash -Algorithm SHA256 <file>` (PowerShell) must give the line found in `SHA256SUMS.txt`.

## 2. Open the demo pack

Unzip `re-play_pack_demo.zip`. On first launch, the setup assistant takes a few clicks; on its last page, click
**Open the Demo project…** and choose the `Demo` folder. Later: *File > Open a project…*.
The pack contains 3 shots (two video versions, an image sequence, an image and its reference), a few images and a `Demo.repl`
edit. The images come from the open movie *Cosmos Laundromat* (© Blender Foundation, cosmoslaundromat.org, CC-BY 4.0).

## 3. Five things to try

1. **Play**: open `PLAN_010` and start playback (space bar); step frame by frame with the arrow keys.
2. **Compare two versions**: on `PLAN_010`, switch from `v001` to `v002` (key **V**: small versions panel, or **Ctrl+↑** / **Ctrl+↓**),
   then try *Difference* or *Wipe*. (The ↑ / ↓ arrows alone go to the previous / next clip.)
3. **Adjust the image**: exposure, gamma, saturation (keys E, G, S + drag); look at the sequence in `PLAN_020`.
4. **Annotate**: draw on a frame (key D), add a note, export the review as a PDF (menu *Pipeline > Review > Export the review as PDF*).
5. **Open a timeline**: *File > Open a timeline (.repl)…* with `Demo.repl`. Then drag **your own media** onto the window.

## 4. Send feedback

Menu **Help > Send feedback…**: a window opens. Write your message
(what you were doing, what happened, what you expected); below it, you see exactly what goes with it: version, system, end of the
log. Paths, your session and machine names, addresses, e-mails and tokens are masked. Then you choose:

- **Copy the message**: plain text for an e-mail or a message;
- **Save a .zip…**: a file (message, diagnostic, and a screenshot of the window if you tick the box) that
  you send yourself. The screenshot shows what is on screen, shots included: only attach it if the image is not confidential.

**re-play sends nothing**: it only copies text or writes a file to your disk. Shorter: **Help > Copy the diagnostic**
(the diagnostic alone, without a message).

**Where to send it**: the `.zip` or the copied text, by e-mail or message, **to the person who gave you the package** (their
contact is in the delivery you received). The "Send feedback" window reminds you at the bottom. The GitHub repository is not
public yet: the "Copy for a GitHub issue" button does not appear in this version.
Always read it through before sending, and never attach a real confidential shot. The `RETOUR_TESTEUR.md` template (in French)
can still be used by hand.

## 5. What is not done

- No automatic update: for a new version, install the new file over the old one (your settings are kept).
- The links with Kitsu, ShotGrid, ftrack and AYON are not tested against real servers in this package.
- The program is not signed (Windows, Mac): warnings at first launch, see above.
- Linux and Mac: less tested than Windows. Tell us anything that gets stuck.
