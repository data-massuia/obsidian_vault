<%*
const inboxPath = "Templates/Task Inbox.md";
const archivePath = "Templates/Task Archive.md";

const inbox = app.vault.getAbstractFileByPath(inboxPath);
const inboxContent = await app.vault.read(inbox);

const lines = inboxContent.split("\n");
const completed = lines.filter(l => l.startsWith("- [x]"));
const remaining = lines.filter(l => !l.startsWith("- [x]"));

const archive = app.vault.getAbstractFileByPath(archivePath);
const archiveContent = await app.vault.read(archive);
await app.vault.modify(archive, archiveContent + "\n" + completed.join("\n"));

await app.vault.modify(inbox, remaining.join("\n"));
%>

- [x] Meditate 2 #td  [repeat:: every day]  [due:: 2026-03-13]  [completion:: 2026-03-06]

- [x] Meditate 2 #td  [repeat:: every day]  [due:: 2026-03-13]  [completion:: 2026-03-06]
- [x] Meditate 2 #td  [repeat:: every day]  [due:: 2026-03-12]  [completion:: 2026-03-06]
- [x] Meditate 2 #td  [repeat:: every day]  [due:: 2026-03-14]  [completion:: 2026-03-06]