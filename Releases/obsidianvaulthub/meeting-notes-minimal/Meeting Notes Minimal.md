---
cssclasses:
  - hide-properties
modified: 2026-06-12
---
```datacorejsx
// ── Configuration ────────────────────────────────────────
// How many days ahead the "Upcoming" list looks.
const DAYS_AHEAD = 14;
// Folders to scan for task/event notes (frontmatter: due / scheduled / start).
const SOURCE_PREFIXES = ["Tasks/"];
// Colors per priority (frontmatter: priority). A `color` frontmatter field overrides this.
const PRIORITY_COLORS = { high: "pink", medium: "orange", low: "blue" };
// Frontmatter `status` values that hide a note from the list.
const DONE_STATUSES = new Set(["done", "completed", "cancelled"]);
// ─────────────────────────────────────────────────────────

const toDate = (value) => {
  if (!value) return null;
  if (value instanceof Date) return isNaN(value.getTime()) ? null : value;
  if (typeof value?.toJSDate === "function") {
    const d = value.toJSDate();
    return isNaN(d.getTime()) ? null : d;
  }
  if (typeof value === "number") {
    const d = new Date(value);
    return isNaN(d.getTime()) ? null : d;
  }
  if (typeof value === "string") {
    const dateOnly = value.match(/^(\d{4})-(\d{2})-(\d{2})$/);
    const d = dateOnly
      ? new Date(Number(dateOnly[1]), Number(dateOnly[2]) - 1, Number(dateOnly[3]))
      : new Date(value);
    return isNaN(d.getTime()) ? null : d;
  }
  return null;
};

const startOfDay = (date) => new Date(date.getFullYear(), date.getMonth(), date.getDate());
const endOfDay = (date) => new Date(date.getFullYear(), date.getMonth(), date.getDate(), 23, 59, 59, 999);
const addDays = (date, days) => new Date(date.getFullYear(), date.getMonth(), date.getDate() + days);

const formatTime = (date) => {
  return date.toLocaleTimeString("en-US", { hour: "numeric", minute: "2-digit" }).replace(":00", "");
};

const formatShortDate = (date) => {
  return date.toLocaleDateString("en-US", { month: "short", day: "numeric" });
};

const formatWeekday = (date) => {
  return date.toLocaleDateString("en-US", { weekday: "short" });
};

const formatLabel = (date, today) => {
  const diff = Math.round((startOfDay(date).getTime() - today.getTime()) / 86400000);
  if (diff < 0) return formatShortDate(date);
  if (diff === 0) return formatTime(date);
  if (diff === 1) return "Tomorrow " + formatTime(date);
  if (diff < 7) return formatWeekday(date) + " " + formatTime(date);
  return formatShortDate(date);
};

return function UpcomingEvents() {
  const [revision, setRevision] = dc.useState(0);

  dc.useEffect(() => {
    const refresh = () => setRevision((current) => current + 1);
    const vaultRefs = [
      app.vault.on("create", refresh),
      app.vault.on("delete", refresh),
      app.vault.on("rename", refresh),
      app.vault.on("modify", refresh),
    ];
    const metadataRefs = [
      app.metadataCache.on("changed", refresh),
      app.metadataCache.on("resolved", refresh),
    ];

    return () => {
      vaultRefs.forEach((ref) => app.vault.offref(ref));
      metadataRefs.forEach((ref) => app.metadataCache.offref(ref));
    };
  }, []);

  const items = dc.useMemo(() => {
    const today = startOfDay(new Date());
    const end = endOfDay(addDays(today, DAYS_AHEAD));

    return app.vault.getMarkdownFiles()
      .filter((file) => SOURCE_PREFIXES.some((prefix) => file.path.startsWith(prefix)))
      .map((file) => {
        const fm = app.metadataCache.getFileCache(file)?.frontmatter ?? {};
        const status = String(fm.status ?? "").toLowerCase();
        if (DONE_STATUSES.has(status)) return null;

        const date = toDate(fm.due) || toDate(fm.scheduled) || toDate(fm.start);
        if (!date || date.getTime() > end.getTime()) return null;

        const priority = String(fm.priority ?? "medium");
        const color = fm.color
          ? String(fm.color).replace(/"/g, "")
          : (PRIORITY_COLORS[priority] ?? "purple");

        return {
          title: fm.title ?? file.basename,
          date,
          color,
          path: file.path,
        };
      })
      .filter(Boolean)
      .sort((a, b) => a.date.getTime() - b.date.getTime())
      .map((item) => ({ ...item, label: formatLabel(item.date, today) }));
  }, [revision]);

  const openItem = (path) => app.workspace.openLinkText(path, "", false);

  return (
    <div className="upcoming-events">
      <details open>
        <summary className="ue-header">Upcoming</summary>
        <div className="ue-body">
          {!items.length ? (
            <div className="ue-empty">No upcoming events.</div>
          ) : items.map((item) => (
            <div
              key={item.path}
              className="ue-event"
              data-color={item.color}
              onClick={() => openItem(item.path)}
              role="button"
              tabIndex={0}
              onKeyDown={(event) => {
                if (event.key === "Enter" || event.key === " ") {
                  event.preventDefault();
                  openItem(item.path);
                }
              }}
            >
              <div className="ue-icon"></div>
              <div className="ue-event-title">{item.title}</div>
              <div className="ue-event-time">{item.label}</div>
            </div>
          ))}
        </div>
      </details>
    </div>
  );
}
```

```datacorejsx
// ── Configuration ────────────────────────────────────────
// Folder where new meeting notes created from a recording are stored.
const NOTES_FOLDER = "Meeting Notes";
// Audio file extensions to list.
const AUDIO_EXTENSIONS = ["mp3", "wav", "m4a", "webm", "ogg"];
// ─────────────────────────────────────────────────────────

function AudioManager() {
  const [recordings, setRecordings] = dc.useState([]);
  const [collapsed, setCollapsed] = dc.useState({});

  const toggleBucket = (label) => {
    setCollapsed(prev => ({ ...prev, [label]: !prev[label] }));
  };

  const fetchRecordings = () => {
    setRecordings(app.vault.getFiles().filter(f => AUDIO_EXTENSIONS.includes(f.extension.toLowerCase())));
  };

  const openInNewTab = async (file) => {
    const leaf = app.workspace.getLeaf('tab');
    await leaf.openFile(file);
    app.workspace.setActiveLeaf(leaf, { focus: true });
  };

  dc.useEffect(() => {
    fetchRecordings();
    const r1 = app.vault.on('create', fetchRecordings);
    const r2 = app.vault.on('delete', fetchRecordings);
    const r3 = app.vault.on('rename', fetchRecordings);
    const r4 = app.vault.on('modify', fetchRecordings);

    return () => {
      app.vault.offref(r1);
      app.vault.offref(r2);
      app.vault.offref(r3);
      app.vault.offref(r4);
    };
  }, []);

  const buckets = dc.useMemo(() => {
    const now = new Date();
    const today = new Date(now.getFullYear(), now.getMonth(), now.getDate());
    const yesterday = new Date(today.getTime() - 86400000);
    const weekStart = new Date(today.getTime() - today.getDay() * 86400000);
    const lastWeekStart = new Date(weekStart.getTime() - 7 * 86400000);

    const sorted = [...recordings].sort((a, b) => b.stat.ctime - a.stat.ctime);
    const groups = [];
    const byKey = {};

    for (const f of sorted) {
      const d = new Date(f.stat.ctime);
      let key, label;

      if (d >= today) {
        key = 'today';
        label = 'Today';
      } else if (d >= yesterday) {
        key = 'yesterday';
        label = 'Yesterday';
      } else if (d >= weekStart) {
        key = 'this-week';
        label = 'This week';
      } else if (d >= lastWeekStart) {
        key = 'last-week';
        label = 'Last week';
      } else {
        key = d.getFullYear() + '-' + d.getMonth();
        label = d.toLocaleDateString('en-US',
          d.getFullYear() !== now.getFullYear()
            ? { month: 'long', year: 'numeric' }
            : { month: 'long' }
        );
      }

      if (!byKey[key]) {
        byKey[key] = { label, files: [] };
        groups.push(byKey[key]);
      }

      byKey[key].files.push(f);
    }

    return groups;
  }, [recordings]);

  const handlePlay = async (file) => {
    await openInNewTab(file);
  };

  const handleDelete = async (file) => {
    if (confirm('Delete "' + file.name + '"?')) {
      await app.vault.trash(file, true);
      fetchRecordings();
    }
  };

  const handleRename = (e, file) => {
    e.stopPropagation();
    app.fileManager.promptForFileRename(file);
  };

  const handleCreateNote = async (file) => {
    const now = new Date();
    const dateStr = now.toISOString().split('T')[0];
    const timeStr = now.toTimeString().split(' ')[0].replace(/:/g, '-');

    const baseName = file.name.replace(/\.[^.]+$/, '');
    const title = baseName || ('Audio Note ' + dateStr + ' ' + timeStr);

    const path = NOTES_FOLDER + '/' + title + '.md';

    const content =
`---
date: ${dateStr}
created: ${now.toISOString()}
audio: "${file.path}"
tags: [meeting-note, audio-note]
---

# ${title}

## Recording

![[${file.path}]]

## Notes

-
`;

    try {
      const folders = NOTES_FOLDER.split('/');
      let currentPath = '';
      for (const f of folders) {
        currentPath = currentPath ? `${currentPath}/${f}` : f;
        if (!await app.vault.adapter.exists(currentPath)) {
          await app.vault.createFolder(currentPath);
        }
      }

      let noteFile;

      if (await app.vault.adapter.exists(path)) {
        noteFile = app.vault.getAbstractFileByPath(path);
        new Notice('Opening existing note.');
      } else {
        noteFile = await app.vault.create(path, content);
        new Notice('Created "' + title + '"');
      }

      await openInNewTab(noteFile);
      return noteFile;

    } catch (e) {
      new Notice('Error: ' + e.message);
      return null;
    }
  };

  return (
    <div className="audio-mgr">
      <div className="am-body">
        {buckets.length === 0 && <div className="am-empty">No recordings yet.</div>}

        {buckets.map((group, i) => {
          const isCollapsed = collapsed[group.label];

          return (
            <div className={`am-bucket ${isCollapsed ? 'is-collapsed' : ''}`} key={i}>
              <div className="am-bucket-label" onClick={() => toggleBucket(group.label)}>
                <span className="am-bucket-arrow">{isCollapsed ? '▶' : '▼'}</span>
                {group.label}
              </div>

              {!isCollapsed && (
                <div className="am-bucket-items">
                  {group.files.map(file => (
                    <RecordingRow
                      key={file.path}
                      file={file}
                      onPlay={handlePlay}
                      onDelete={handleDelete}
                      onRename={handleRename}
                      onCreateNote={handleCreateNote}
                      openInNewTab={openInNewTab}
                    />
                  ))}
                </div>
              )}
            </div>
          );
        })}
      </div>
    </div>
  );
}

function RecordingRow({ file, onPlay, onDelete, onRename, onCreateNote, openInNewTab }) {
  const [hasNote, setHasNote] = dc.useState(null);

  const findLinkedNote = async () => {
    const checks = [
      `![[${file.name}]]`,
      `![[${file.path}]]`,
      `audio: "${file.path}"`,
      `audio: ${file.path}`,
      file.path
    ];

    for (const md of app.vault.getMarkdownFiles()) {
      try {
        const text = await app.vault.cachedRead(md);

        if (checks.some(check => text.includes(check))) {
          setHasNote(md);
          return;
        }
      } catch (e) {}
    }

    setHasNote(false);
  };

  dc.useEffect(() => {
    findLinkedNote();
  }, [file.path]);

  const handleNoteAction = async (e) => {
    e.stopPropagation();

    if (hasNote && hasNote !== false) {
      await openInNewTab(hasNote);
    } else {
      const newNote = await onCreateNote(file);
      if (newNote) setHasNote(newNote);
    }
  };

  const baseName = file.name.replace(/\.[^.]+$/, '');
  const noteAttached = hasNote && hasNote !== false;
  const timeStr = moment(file.stat.ctime).format("h:mm A").replace(":00", "");

  return (
    <div className="am-item" onClick={() => onPlay(file)}>
      <div className="am-item-icon">
        {noteAttached ? (
          <svg viewBox="0 0 24 24">
            <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
            <polyline points="14 2 14 8 20 8"></polyline>
            <line x1="9" y1="13" x2="15" y2="13"></line>
            <line x1="9" y1="17" x2="13" y2="17"></line>
          </svg>
        ) : (
          <svg viewBox="0 0 24 24">
            <line x1="6" y1="9" x2="6" y2="15"></line>
            <line x1="10" y1="6" x2="10" y2="18"></line>
            <line x1="14" y1="6" x2="14" y2="18"></line>
            <line x1="18" y1="9" x2="18" y2="15"></line>
          </svg>
        )}
      </div>

      <span className="am-item-title">{baseName}</span>
      <span className="am-item-time">{timeStr}</span>

      <div className="am-item-actions">
        <button
          className="am-icon-btn"
          onClick={handleNoteAction}
          title={noteAttached ? 'Open note in new tab' : 'Create note in new tab'}
        >
          <svg viewBox="0 0 24 24">
            {noteAttached
              ? <>
                  <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                  <polyline points="14 2 14 8 20 8"></polyline>
                  <path d="M16 13H8"></path>
                  <path d="M16 17H8"></path>
                  <path d="M10 9H8"></path>
                </>
              : <>
                  <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"></path>
                  <polyline points="14 2 14 8 20 8"></polyline>
                  <path d="M12 18v-6"></path>
                  <path d="M9 15h6"></path>
                </>
            }
          </svg>
        </button>

        <button className="am-icon-btn" onClick={(e) => onRename(e, file)} title="Rename">
          <svg viewBox="0 0 24 24">
            <path d="M17 3a2.85 2.83 0 1 1 4 4L7.5 20.5 2 22l1.5-5.5L17 3z"></path>
          </svg>
        </button>

        <button
          className="am-icon-btn am-delete"
          onClick={(e) => {
            e.stopPropagation();
            onDelete(file);
          }}
          title="Delete"
        >
          <svg viewBox="0 0 24 24">
            <polyline points="3 6 5 6 21 6"></polyline>
            <path d="M19 6v14a2 2 0 0 1-2 2H7a2 2 0 0 1-2-2V6m3 0V4a2 2 0 0 1 2-2h4a2 2 0 0 1 2 2v2"></path>
          </svg>
        </button>
      </div>
    </div>
  );
}

return <AudioManager />;
```
