# Making Copilot Agents Produce On‑Brand Content

**A HackPack reference guide** — how to get Microsoft 365 Copilot (and agents you build with **Agent Builder**) to follow a **style guide** and use your **PowerPoint templates**, plus a clear map of what Agent Builder *can* and *can't* do.

> **TL;DR**
> - **Style guide (tone, terminology, format): ✅ easy.** Add it as a **Knowledge** source and enforce it in **Instructions**.
> - **A real `.potx` PowerPoint template: ⚠️ not from an Agent Builder agent itself.** Use **PowerPoint Copilot** with a SharePoint **Organization Asset Library (OAL)** (admin setup) or **Copilot Cowork** (attach a personal `.potx`).
> - **Writing back to M365 (Planner tasks, Teams posts, calendar): ❌ not in Agent Builder.** That requires **Copilot Studio** actions or a pro‑code **Agents Toolkit** API plugin.

---

## 1. What Agent Builder actually gives you

An agent built with **Agent Builder** in Microsoft 365 Copilot is a **declarative agent** = **Instructions** + **Knowledge** + a few built‑in **Capabilities**.

| Part | What it does | Good for |
|---|---|---|
| **Instructions** | Natural‑language rules for behavior, tone, format | Enforcing a style guide, response shape |
| **Knowledge** | Read‑only grounding: SharePoint/OneDrive files, Copilot (Graph) connectors, web | Feeding the agent your style guide, reference docs |
| **Capabilities** | Built‑in toggles: *Create documents/charts/code*, *Create images* | Generating drafts, decks, visuals |

**The ceiling:** Agent Builder has **no Actions**. Microsoft's own doc says: *"If you need more advanced capabilities like **Actions to integrate external services, use Microsoft Copilot Studio**."* So an Agent Builder agent can **read/ground and generate**, but it **cannot write** to Planner/Teams/calendar and **cannot bind** a specific `.potx` template.

---

## 2. Enforce a style guide  ✅

Two levers, used together:

**Step 1 — Add the style guide as Knowledge.** Put the Word/PDF style guide somewhere the agent can read (SharePoint or OneDrive), then add it under **Knowledge**.

**Step 2 — Restate the non‑negotiables in Instructions and reference the doc.** Per Microsoft guidance: *always specify tone, verbosity, and output format*, write in **Markdown**, and **bold** critical rules.

```markdown
## Style rules (always apply)
- **Always follow the "Contoso Content Style Guide"** knowledge file for voice, terminology, and formatting.
- **Tone:** professional, concise, active voice.
- **Headings:** Title Case. **Never** use exclamation points.
- Approved terms: "teammate" (not "employee"); "Copilot" (not "the AI").
- For decks, follow this section order: Title → Agenda → Context → Recommendation → Next Steps.
```

---

## 3. Use your PowerPoint template  ⚠️ (three routes)

| Route | Applies your `.potx`? | Who sets it up | Notes |
|---|---|---|---|
| **Agent Builder agent** | ❌ No | — | No mechanism to bind a template |
| **PowerPoint Copilot + SharePoint OAL** | ✅ Yes | **Admin** (tenant‑wide) | Templates appear in the Copilot "create new presentation" picker |
| **Copilot Cowork** | ✅ Yes | **You** (personal) or admin | Attach a `.potx` (or drop it in OneDrive root) and tell Cowork to use it — no admin needed |

**What carries through from a template** (verified by Microsoft for Cowork): theme palette, brand fonts, logos, **all named layouts + slide master**, and slide geometry/placeholder positions — Cowork rewrites the *text* and leaves the *design* intact.

---

## 4. Admin setup — SharePoint Organization Asset Library (for templates)

One‑time, via the **SharePoint Online Management Shell** (requires **SharePoint Administrator**). There is no admin‑center UI toggle for this.

1. **Pick/create one SharePoint site** for org assets. ⚠️ *All* org asset libraries must live on the **same site**.
2. **Create a document library** and **upload your templates** — PowerPoint `.potx` (Word `.dotx`, Excel `.xltx`).
3. Install the latest **[SharePoint Online Management Shell](https://go.microsoft.com/fwlink/p/?LinkId=255251)** (uninstall older versions first).
4. Connect as a SharePoint Administrator:
   ```powershell
   Connect-SPOService -Url https://<tenant>-admin.sharepoint.com
   ```
5. Register the library as a **template** library (`-OrgAssetType OfficeTemplateLibrary` is the key — without it, it defaults to an image library):
   ```powershell
   Add-SPOOrgAssetsLibrary `
     -LibraryUrl  https://contoso.sharepoint.com/sites/branding/Templates `
     -ThumbnailUrl https://contoso.sharepoint.com/sites/branding/Templates/contosologo.jpg `
     -OrgAssetType OfficeTemplateLibrary `
     -CdnType Private
   ```
   > Want both approved **images** *and* **templates**? Use `-OrgAssetType ImageDocumentLibrary,OfficeTemplateLibrary`.
   > Convert an existing library later: `Set-SPOOrgAssetsLibrary -LibraryUrl <url> -OrgAssetType OfficeTemplateLibrary`.

**Then, as an end user:** PowerPoint → **New → Create with Copilot** → choose a template from **your organization's collection**. The deck is generated on‑brand.

### Requirements & gotchas
- **`OrgAssetType` values:** `ImageDocumentLibrary`, `OfficeTemplateLibrary`, `OfficeFontLibrary`, `BrandKitLibrary` (combinable).
- Up to **30** libraries, all on the **same site**; **allow up to 24 hours** to appear.
- Users need **read** on your org **root site**.
- PowerPoint **on the web** requires **Office 365 E3/E5**; desktop needs **Microsoft 365 Apps v2002+**.
- Publishing a `.potx` to the OAL feeds **both** PowerPoint Copilot's picker **and** Copilot Cowork.

---

## 5. Writing back to M365 (Planner, Teams, Calendar)  ❌ in Agent Builder

Any **write** action has the same ceiling as templates — not available in Agent Builder. Options one tier up:

| Need | Path |
|---|---|
| Add a **Planner** task | Copilot Studio → **Planner connector** action, *or* Agents Toolkit API plugin over Microsoft Graph `POST /planner/tasks` |
| Post a **Teams** channel message | Copilot Studio → **Teams connector** ("Post message…"), *or* Graph `POST /teams/{id}/channels/{id}/messages` |
| Read **Teams "Updates"** submissions | *Updates App (Microsoft 365)* connector is **trigger‑only** (no read action) — trigger → persist to Dataverse/SharePoint → read |

---

## 6. Official documentation

**On‑brand content / templates**
- Create an organization assets library — <https://learn.microsoft.com/en-us/sharepoint/organization-assets-library>
- `Add-SPOOrgAssetsLibrary` cmdlet — <https://learn.microsoft.com/en-us/powershell/module/sharepoint-online/add-spoorgassetslibrary>
- Connect organizational asset libraries to PowerPoint (Brand Images) — <https://learn.microsoft.com/en-us/sharepoint/connect-organizational-asset-libraries-to-copilot>
- Create a new presentation with Copilot in PowerPoint — <https://support.microsoft.com/office/create-a-new-presentation-with-copilot-in-powerpoint-3222ee03-f5a4-4d27-8642-9c387ab4854d>
- Use brand templates with Copilot Cowork — <https://learn.microsoft.com/en-us/microsoft-365/copilot/cowork/cowork-brand-templates>

**Agent Builder / declarative agents**
- Agent Builder in Microsoft 365 Copilot — <https://learn.microsoft.com/en-us/microsoft-365-copilot/extensibility/agent-builder>
- Write effective instructions for declarative agents — <https://learn.microsoft.com/en-us/microsoft-365/copilot/extensibility/declarative-agent-instructions>

---

*Part of the HackPack — Agents Hackathon Workshop. All facts sourced from official Microsoft Learn / Support documentation.*
