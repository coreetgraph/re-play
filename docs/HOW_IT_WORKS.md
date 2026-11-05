# How re-play works

*Version française : [COMMENT_CA_MARCHE.md](COMMENT_CA_MARCHE.md).*

re-play is the place where shots are **viewed, checked, decided on and passed along**. It does not make images (Nuke, Blender
and ComfyUI do that) and it does not replace a production tracker (Kitsu, ShotGrid). It runs on your computer, with no
account, no server and no built-in AI. Two rules apply everywhere: **original media are never modified**, and **nothing leaves
the computer without a click** in a confirmation window.

A shot's journey through re-play:

```
 files received ──► INGEST ──► CHECK ──► REVIEW ──► PUBLISH / DELIVER
  (Nuke, AI,        file and    look at   notes,     Kitsu / ShotGrid,
   a colleague)     QC          the image verdict    dated delivery folder
                       ▲                                   │
                       └────────── INBOX ◄─────────────────┘
                          (feedback, retakes, AI proposals)
```

Version described: 0.9.31 (November 2026).

---

## 1. First: how re-play understands your folders (the profile)

re-play needs to know **how projects are laid out on disk**: where a project is, where its shots, versions and stages
(Layout, 3D, Comp…) are. That is the job of the **studio profile**, a small settings file.

### Three ways of working

| Profile | For whom | What re-play understands |
|---|---|---|
| **Blank** (simple player) | one person, loose folders | nothing imposed: you open folders and play media. No shots, no automatic versions. |
| **Prism** | a studio that organises its projects with Prism | the whole Prism tree: sequences, shots, stages, versions, statuses. |
| **Custom profile** ("Guessed — …") | a studio with its own layout | re-play looks at an existing project, guesses its structure, shows it, and saves it as a profile. |

### Showing a project to re-play ("Analyze a project")

*Preferences > Projects*, section "Folder to analyze" (or page 2 of the first-launch assistant, "Show one of my
projects…"):

1. Pick the folder of an existing project, then **Analyze**.
2. re-play shows **what it understood**: one line per folder level (sequence, shot, stage, version…), with examples. A level
   guessed wrong can be corrected by hand.
3. It lists the **media found** and what it **noticed** (shots without a V000, missing or duplicate versions, "v" and "V"
   mixed, the same stage spelled several ways: comp / compo / compositing…). These are suggestions: nothing is changed.
4. **Use this structure**: the structure becomes a profile in your preferences. **Nothing is written in the project.**

Folders where re-play looks for projects are added in *Preferences > Projects > Project folders*.

### What a profile contains

A profile is a JSON file. For Prism, the key lines say:

- **what makes a project**: a folder containing `03_Production`, `PROJET.md` or `00_Pipeline/pipeline.json`;
- **the tree**: `03_Production/Shots/{sequence}` → `{plan}` (the shot) → `Renders/{famille}/{identifiant}` → `{version}` → `[rgb]`;
- **the version format**: `v###` (v001, v002…);
- **the stage rule**: the stage is the start of the identifier, up to the first hyphen.

Example, this path:

```
D:/PROJECTS/DEMO/03_Production/Shots/SEQ/SEQ_0010/Renders/external/Comp-alpha/v003/rgb/SEQ_0010_comp_v003.0001.exr
```

is read like this:

| Part | Meaning |
|---|---|
| `D:/PROJECTS/DEMO` | the project (it contains `03_Production`) |
| `SEQ` | the sequence |
| `SEQ_0010` | the shot |
| `external` | the render family |
| `Comp-alpha` | the identifier… |
| `Comp` | …whose **stage** is Comp (before the hyphen) |
| `v003` | the version |

**The hyphen rule**: `Comp-alpha`, `Comp-nograin` and `Comp` are all the same stage, **Comp**; `3D-beauty` is the stage **3D**.
Only the profile's separator counts (a hyphen by default): `Comp_alpha` or `Compo-alpha` make a separate stage. This rule is
what fills the POST-PROD layers of the timeline (V000, Layout, 3D, Comp…).

### Several computers, several systems

The same share does not have the same name everywhere (`Z:\` on Windows, `/Volumes/projects/` on Mac, `/mnt/projects/` on
Linux). *Preferences > Storage > Mappings between systems* tells re-play these paths are the same: a review or timeline made on
one computer opens on another. Local copies are set up there too (working away from the studio on a copy of a NAS project).

### Where re-play keeps its own settings

| | Windows | Mac | Linux |
|---|---|---|---|
| Preferences, custom profiles, AI log | `%APPDATA%\re-play\` | `~/Library/Application Support/re-play/` | `~/.config/re-play/` |
| Cache (can be changed: *Preferences > Storage*) | `%LOCALAPPDATA%\re-play\cache` | `~/Library/Caches/re-play` | `~/.cache/replay` |

Custom profiles are in the `profils/` subfolder of the preferences (*Preferences > Projects > Open the profiles folder*).

---

## 2. Ingest: filing incoming media

*Tools > Ingest: file the shots…* (Ctrl+Shift+I), or right-click a shot in the library > *Ingest into this shot…*.

Ingest takes files from outside (a render, an AI output, something a colleague sent) and **copies** them to the right place in
the project, with the right name and a new version number.

1. Choose the project, then add the files (button or drag and drop).
2. For each file, re-play guesses the **shot** from its name or path (`TRA0020` → `TRA_0020`), and suggests the stage, the next
   version and the **final name**. Everything can be corrected in the table; a missing shot can be created on the spot.
3. A **shared note** keeps the context (prompt, setting, who sent it).
4. **Declare** opens a "source → destination" summary. Nothing is written yet. **Back** to correct, **Declare** to copy.

What happens next:

- Files are **copied, never moved**, and **nothing is written in the source folder**.
- The copy goes into a new version (`v003` if `v002` exists); a name already taken gets `_2`, never overwritten.
- A `versioninfo.json` sheet keeps the origin and the note; the project keeps a list of declarations
  (`.re-play/declarations.json`).
- The declarations window lets you correct or cancel. Cancelling deletes nothing: the copy goes to the project's
  `_corbeille_replay/` folder, where it can be restored.

### Ingest quality control (QC)

While the table is being filled, re-play checks each file in the background (a badge in the QC column: **ok**, **to check**,
**blocking**): missing or badly numbered frames, black or duplicate frames, invalid pixels in EXRs, frame rate or resolution
different from the project, unexpected duration, missing audio, name outside the naming convention. Nothing is refused
outright: if there is a blocking issue, re-play asks for confirmation. The report (`QC_<name>.md`) is filed with the version.

### The V000

The **V000** is the shot **without effects**, cut from the original edit with its handles. It is the reference: every version
is compared to it, and it serves as the source until a working version exists. It is **read-only**; work starts at v001. It is
made by cutting an edit (conform, *Distribute* window), not by ingest.

---

## 3. Check: looking closely at the image

The **Check** tab of the review panel (Shift+R opens the panel). It is a check **by eye**, and it changes no file:

- **Adjust the image**: hold E (exposure), G (gamma), S (saturation) or N (levels) and drag; *Reset all*.
- **Grain and clipping**: *Grain view*; *Clipping* (red stripes = blown whites, blue = crushed blacks); *False color* (T);
  *EXR windows* (Shift+T).
- **Against the previous version**: *Difference* |A − B| (what changed stands out), *Wipe*, back to the normal image (W).
- **Against the V000**: the same comparison with the shot without effects, to see exactly what the effect did.

---

## 4. Review

A review gathers the versions to judge, the notes, the drawings and the verdicts. To start one: the **Do a review** card on
the start page, the **Create the review** button in the panel (with the media loaded), or *Pipeline > Review > Open a review…*.

The panel follows **4 steps**, shown at the top (green = done, blue = current, click to go there):

1. **Versions**: which version to look at? The shot's versions are at the top, with A / B comparison (wipe, side by side,
   difference). A version watched for more than a second is marked "seen".
2. **Notes**: one note per version, with the drawn image if needed (D key: freehand, arrow, frame, text). The thread keeps
   the whole history.
3. **Verdict**: *Approve*, *Retake*, *Select* or *Note only*; the project's other statuses as small badges.
4. **Delivery**: the summary ("2 approved, 1 retake"), then:
   - **Publish…**: sends verdicts and notes to Kitsu or ShotGrid. A window shows everything first (shot, version, status set,
     note, image); nothing leaves before the *Publish* button. For a shot to retake, you choose who for: a team member, the AI
     assistant, or later;
   - **Prepare delivery…**: builds a dated delivery folder for the client or the editor
     (`05_Echanges/OUT/<date>_<recipient>`), as is or transcoded, with on-image info, slate or watermark if wanted. re-play
     prepares the folder; you send it yourself.

Other tabs in the panel: *Check* (above), *History*, *Reviews* (the shot's past reviews, double-click to reopen), *Info*,
*AI*.

**Where it is saved**: in the shot's folder, `_revue/<date>.json` for the review, and `_revue/retours.jsonl`, a log where
each action is added without ever rewriting the previous one. *Pipeline > Review* also exports the review as a **PDF** (one
page per note, with the drawn image) or prepares an email draft.

### The inbox

Everything that arrives from outside while you work goes through the **Inbox** (badge at the top, *File > Inbox…*): a
`replay://` link, a file sent by a script, a retake published by a supervisor, a proposal from the AI. **Nothing replaces the
open session**: you choose to open it in a new tab, add it to the current one, or dismiss it.

---

## 5. AI in re-play (MCP server)

re-play **contains no AI**. It can, however, **let an AI work with it** (Claude, Codex, Gemini, a local AI…) through a small
server in the **MCP** format, a standard these assistants know how to use.

### What the AI can do

- **Read**: projects, shots, versions, notes, client feedback, deliveries, the profile, and a description of a media file.
- **Propose**: drop a new version (filed according to the profile, never overwriting), send files to look at, say that it is
  taking on a retake and how far along it is, ask for approval.

### What it cannot do

Give a verdict, publish to Kitsu or ShotGrid, send anything to the client, delete, change settings. These tools do not exist
for it. **Everything it drops arrives in the Inbox**, and a human looks and decides.

Other safeguards:

- It only sees the project folders re-play knows about, not the rest of the computer.
- The server does not listen on the network: the assistant starts it on the same machine.
- Every call is written to a log (`mcp/journal.jsonl` in the preferences folder), with passwords masked.
- A paid online service (for example Comfy Cloud) or going over the credit limits requires explicit approval.
- The **AI light**, at the top of the window: **grey** = the AI only reads, **green** = allowed to drop proposals, **orange** =
  it has just worked (in the last 2 minutes). Click it to see the last call and open the log.

### Setting it up at home

1. **Open re-play** > *Preferences > AI · MCP*.
2. **Decide what it is allowed to do**:
   - nothing ticked: the AI can only read;
   - **MCP on**: it can drop proposals in the Inbox;
   - **Read-only**: even when on, it cannot write anything;
   - **The AI sees the images** (unticked by default) and the credit limits.
3. **(Recommended) Prepare the context**: a short text that explains your studio, project and habits to the AI
   (`CONTEXTE_IA.md`). The assistant pre-fills it; amounts, logins and tokens are removed automatically.
   *What does the AI know about me?* shows what it will read.
4. **Connect the assistant**: the page gives a ready-made command to copy.
   - **Claude Code**: paste the line in a terminal. It looks like:
     ```
     claude mcp add --scope user re-play -- "<re-play folder>\re-play.exe" --mcp
     ```
     (the page writes the real path of re-play on this computer).
   - **Other client** (Claude Desktop, Codex, Gemini CLI, Cline, LM Studio, Open WebUI…): copy the "Other client (JSON)"
     block into that software's MCP settings:
     ```json
     { "mcpServers": { "re-play": { "command": "<path to re-play>", "args": ["--mcp"] } } }
     ```
5. **Check**: open a **new** session of the assistant and ask "List my re-play projects". Then "Approve shot SEQ_0010": it
   must answer that it is not allowed to. The light turns orange while it works.

The projects the AI can see are those in *Preferences > Projects*: if it cannot find a project, the project is not in a known
project folder.

**Local AI** (nothing leaves the computer): a 7 to 8 billion parameter model with "tools" support in Ollama, connected through
an MCP client (Open WebUI, Goose, Cline, LM Studio), with the same JSON block.

On Windows, re-play can also **wake the assistant up on its own** when a retake is assigned to it (*Automatic start*: Claude
Code, Codex, Gemini CLI or another command, then *Install the watcher*).

---

## Summary

| Step | Where | What is written |
|---|---|---|
| Understanding projects | Preferences > Projects | a profile, in the preferences (nothing in the project) |
| Ingest | Tools > Ingest: file the shots… | a copy in a new version, its sheet, the QC report |
| Check | review panel, Check tab | nothing |
| Review | review panel, 4 steps | `_revue/` in the shot's folder |
| Publish / Deliver | step 4 of the review | in Kitsu / ShotGrid or in a delivery folder, **after confirmation** |
| AI | Preferences > AI · MCP | its proposals, in the Inbox |

Questions, bugs, ideas: the [Core et Graph Discord](https://discord.gg/BDAKUGz5jp) (channels `re-play-help`, `re-play-bugs`,
`re-play-ideas`), the [feedback form](https://tally.so/r/WJ25qL), or re-play@coreetgraph.com.
