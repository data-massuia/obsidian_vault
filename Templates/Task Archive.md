<%*
const inboxPath = "Task Inbox.md";
const archivePath = "Task Archive.md";

const inbox = await app.vault.getAbstractFileByPath(inboxPath);
const inboxContent = await app.vault.read(inbox);

const lines = inboxContent.split("\n");
const completed = lines.filter(l => l.startsWith("- [x]"));
const remaining = lines.filter(l => !l.startsWith("- [x]"));

// Append to archive
const archive = await app.vault.getAbstractFileByPath(archivePath);
const archiveContent = await app.vault.read(archive);
await app.vault.modify(archive, archiveContent + "\n" + completed.join("\n"));

// Update inbox
await app.vault.modify(inbox, remaining.join("\n"));
%>
