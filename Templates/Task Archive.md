<%*
const inboxPath = "Templates/Task Inbox.md";
const archivePath = "Templates/Task Archive.md";

const inbox = app.vault.getAbstractFileByPath(inboxPath);
const archive = app.vault.getAbstractFileByPath(archivePath);

const inboxContent = await app.vault.read(inbox);
const archiveContent = await app.vault.read(archive);

const lines = inboxContent.split("\n");
const completed = lines.filter(l => l.match(/^- \[x\]/i));
const remaining = lines.filter(l => !l.match(/^- \[x\]/i) && l.trim() !== "- [ ]");

if (completed.length === 0) {
    new Notice("No completed tasks found!");
} else {
    const today = new Date().toISOString().split("T")[0];
    const toArchive = completed.map(l => l + " (archived: " + today + ")");
    
    const newArchiveContent = archiveContent.trimEnd() + "\n" + toArchive.join("\n") + "\n";
    const newInboxContent = remaining.join("\n") + "\n";
    
    await app.vault.modify(archive, newArchiveContent);
    await app.vault.modify(inbox, newInboxContent);
    new Notice(completed.length + " task(s) archived!");
}
%>
- [x] Meditate 2 #td  [repeat:: every day]  [due:: 2026-03-26]  [completion:: 2026-03-06] (archived: 2026-03-06)
