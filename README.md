App.tsx:
import { useEffect, useRef, useState } from 'react';
import FileUploader from './Components/FileUploader';
import LogTableReactWindow from './Components/LogTableReactWindow';
import { registerLocale } from 'react-datepicker'
import { tr } from 'date-fns/locale';

import './styles/logviewer.css';

registerLocale('tr', tr);

interface LogItem {
  timestamp: string;
  timestamp_iso: string;
  level: string;
  service: string;
  event_type: string;
  message: string;
  source_path: string;
  console: number | null;
  is_service_log: number;
}

interface Filters {
  levels: string[];
  eventTypes: string[];
  files: string[];       
  consoles: number[];
  scope: 'all' | 'service' | 'other';
  from: string | null;
  to: string | null;
  messageContains: string;
  sortDir: 'asc' | 'desc';
}

interface Facets {
  files: string[];        
  consoles: number[];
}

const PAGE_SIZE = 3000;
const DEFAULT_FILTERS: Filters = {
  levels: [], eventTypes: [], files: [], consoles: [], scope: 'all',
  from: null, to: null, messageContains: '', sortDir: 'desc',
};

const buildQuery = (targetPage: number, f: Filters) => {
  const params = new URLSearchParams();
  params.set('page', String(targetPage));
  params.set('limit', String(PAGE_SIZE));
  if (f.levels.length) params.set('levels', f.levels.join(','));
  if (f.eventTypes.length) params.set('eventTypes', f.eventTypes.join(','));
  if (f.files.length) params.set('files', f.files.join(','));
  if (f.consoles.length) params.set('consoles', f.consoles.join(','));
  if (f.scope !== 'all') params.set('scope', f.scope);
  if (f.from) params.set('from', f.from);
  if (f.to) params.set('to', f.to);
  if (f.messageContains) params.set('messageContains', f.messageContains);
  params.set('sortDir', f.sortDir);
  return params.toString();
};

function hasActiveFiltersCheck(f: Filters): boolean {
  return (
    f.levels.length > 0 ||
    f.eventTypes.length > 0 ||
    f.files.length > 0 ||        
    f.consoles.length > 0 ||
    f.scope !== 'all' ||
    !!f.from ||
    !!f.to ||
    !!f.messageContains
  );
}

function App() {
  const [logs, setLogs] = useState<LogItem[]>([]);
  const [page, setPage] = useState(1);
  const [hasMore, setHasMore] = useState(true);
  const [loading, setLoading] = useState(false);
  const [filters, setFilters] = useState<Filters>(DEFAULT_FILTERS);
  const [facets, setFacets] = useState<Facets>({ files: [], consoles: [] });
  const fetchingRef = useRef(false);

  const loadPage = async (
    targetPage: number,
    replace: boolean,
    filtersOverride?: Filters,
  ) => {
    if (fetchingRef.current) return;
    fetchingRef.current = true;
    setLoading(true);
    try {
      // filtersOverride: setFilters state is not updated yet -> async
      // new filter goes through here - to ensure not take the old value from closure 
      const activeFilters = filtersOverride ?? filters;
      const response = await fetch(
        `http://localhost:3000/logs?${buildQuery(targetPage, activeFilters)}`,
      );
      const data = await response.json();

      if (!Array.isArray(data)) {
        console.error('Response is not an array:', data);
        if (replace) setLogs([]);
        return;
      }

      setLogs((prev) => (replace ? data : [...prev, ...data]));
      setHasMore(data.length === PAGE_SIZE);
      setPage(targetPage);
    } catch (error) {
      console.error('Fetch failed:', error);
    } finally {
      setLoading(false);
      fetchingRef.current = false;
    }
  };

  const fetchFacets = async () => {
    try {
      const response = await fetch('http://localhost:3000/facets');
      const data = await response.json();
      if (data && Array.isArray(data.files) && Array.isArray(data.consoles)) {
        setFacets(data);
      }
    } catch (error) {
      console.error('Facets fetch failed:', error);
    }
  };

  const handleNearEnd = () => {
    if (!hasMore || fetchingRef.current) return;
    loadPage(page + 1, false);
  };

  const applyFilters = (newFilters: Filters) => {
    setFilters(newFilters);
    setHasMore(true);
    loadPage(1, true, newFilters);
  };

  const onClickResetDatabase = async () => {
    try {
      await fetch('http://localhost:3000/reset', { method: 'POST' });
      setLogs([]);
      setPage(1);
      setHasMore(true);
      setFilters(DEFAULT_FILTERS);
      setFacets({ files: [], consoles: [] });
    } catch (error) {
      console.error('Reset failed:', error);
    }
  };

  useEffect(() => {
    loadPage(1, true);
    fetchFacets();
  }, []);

  const filtersActive = hasActiveFiltersCheck(filters);

  return (
    <div className="lv-shell">
      <header className="lv-header">
        <div className="lv-brand">
          <h1>Log Viewer</h1>
          <span>console inspector</span>
        </div>
        <div className="lv-spacer" />
        <button className="lv-btn lv-btn-danger" onClick={onClickResetDatabase}>
          Clear logs
        </button>
      </header>

      <FileUploader
        onUploadComplete={() => {
          setHasMore(true);
          loadPage(1, true);
          fetchFacets();
        }}
      />

      <div className="lv-toolbar">
        <span className="lv-page">{logs.length.toLocaleString()} rows uploaded</span>
        {filtersActive && (
          <button className="lv-btn" onClick={() => applyFilters(DEFAULT_FILTERS)}>
            Clear all filters
          </button>
        )}
        {loading && <span className="lv-loading">loading…</span>}
      </div>

      <LogTableReactWindow
        logs={logs}
        onNearEnd={handleNearEnd}
        filters={filters}
        facets={facets}
        onApplyFilters={applyFilters}
        hasActiveFilters={filtersActive}
        onClearFilters={() => applyFilters(DEFAULT_FILTERS)}
      />
    </div>
  );
}

export default App;


index.html:
<!doctype html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/favicon.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>frontend</title>
  </head>
  <body>
    <div id="root"></div>
    <div id="lv-datepicker-portal"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
  
</html>


LogTableReactWindow.tsx:
import { useState } from 'react';
import { List, type RowComponentProps } from 'react-window';
import FilterPopover from './FilterPopover';
import { CheckboxListFilter, SourceFilter, RangeFilter, TextContainsFilter } from './FilterContents';

interface LogItem {
  timestamp: string;
  level: string;
  service: string;
  event_type: string;
  message: string;
  source_path: string;
  console: number | null;
  is_service_log: number;
  file_name: string;   
}

interface Filters {
  levels: string[];
  eventTypes: string[];
  files: string[];       
  consoles: number[];
  scope: 'all' | 'service' | 'other';
  from: string | null;
  to: string | null;
  messageContains: string;
  sortDir: 'asc' | 'desc';
}

interface Facets {
  files: string[];        
  consoles: number[];
}

interface LogTableProps {
  logs: LogItem[];
  onNearEnd: () => void;
  filters: Filters;
  facets: Facets;
  onApplyFilters: (filters: Filters) => void;
  hasActiveFilters: boolean;
  onClearFilters: () => void;
}

const LEVEL_OPTIONS = ['info', 'warning', 'error'];
const EVENT_TYPE_OPTIONS = ['event', 'query', 'command', 'commandResult'];

function stripExtension(fileName: string): string {
  return fileName.replace(/\.(ndjson|json|log)$/i, '');
}

const levelKey = (level: string) => {
  const l = level.toLowerCase();
  if (l === 'error' || l === 'warning' || l === 'info') return l;
  return 'info';
};

function LogRow({
  index,
  style,
  logs,
}: RowComponentProps<{
  logs: LogItem[];
}>) {
  const log = logs[index];
  const lvl = levelKey(log.level);
  const [date, ...rest] = log.timestamp.split(' ');
  const time = rest.join(' ');

  return (
    <div className="lv-vrow" style={style}>
      <div className={`lv-vcell lv-col-timestamp lv-td-stripe lv-stripe-${lvl}`}>
        <span className="lv-ts-date">{date} </span>
        <span className="lv-ts-time">{time}</span>
      </div>
      <div className="lv-vcell lv-col-level">
        <span className={`lv-badge lv-badge-${lvl}`}>{log.level}</span>
      </div>
      <div className="lv-vcell lv-col-service lv-cell-clip lv-service" title={log.file_name}>
        {stripExtension(log.file_name)}
      </div>
      <div className="lv-vcell lv-col-event lv-cell-clip lv-event" title={log.event_type}>
        {log.event_type}
      </div>
      <div className="lv-vcell lv-col-message lv-msg" title={log.message}>
        {log.message}
      </div>
      <div className="lv-vcell lv-col-source lv-src" title={log.source_path}>
        {log.source_path}
      </div>
    </div>
  );
}

const FETCH_THRESHOLD = 300;

type OpenFilterKey = 'timestamp' | 'level' | 'service' | 'eventType' | 'message' | 'source' | null;

const LogTableReactWindow = ({
  logs, onNearEnd, filters, facets, onApplyFilters, hasActiveFilters, onClearFilters,
}: LogTableProps) => {
  const [openFilter, setOpenFilter] = useState<OpenFilterKey>(null);
  const close = () => setOpenFilter(null);

  return (
    <div className="lv-tablewrap">
      {/* Header her zaman render edilir — 0 satır dönse bile filtrelere erişim kaybolmaz */}
      <div className="lv-vheader">
        <div className="lv-vheader-cell lv-col-timestamp lv-vheader-filterable">
          <div className="lv-vheader-inner">
            <button
              className={`lv-vheader-btn ${filters.from || filters.to ? 'lv-vheader-btn-active' : ''}`}
              onClick={() => setOpenFilter(openFilter === 'timestamp' ? null : 'timestamp')}
            >
              Timestamp
            </button>
            <button
              className="lv-sort-btn"
              title={filters.sortDir === 'asc' ? 'Ascending — click for descending' : 'Descending — click for ascending'}
              onClick={() => onApplyFilters({ ...filters, sortDir: filters.sortDir === 'asc' ? 'desc' : 'asc' })}
            >
              {filters.sortDir === 'asc' ? '▲' : '▼'}
            </button>
          </div>
          <FilterPopover open={openFilter === 'timestamp'} onClose={close}>
            <RangeFilter
              value={{ from: filters.from, to: filters.to }}
              onApply={({ from, to }) => onApplyFilters({ ...filters, from, to })}
              onClose={close}
            />
          </FilterPopover>
        </div>

        <div className="lv-vheader-cell lv-col-level lv-vheader-filterable">
          <button
            className={`lv-vheader-btn ${filters.levels.length ? 'lv-vheader-btn-active' : ''}`}
            onClick={() => setOpenFilter(openFilter === 'level' ? null : 'level')}
          >
            Level
          </button>
          <FilterPopover open={openFilter === 'level'} onClose={close}>
            <CheckboxListFilter
              title="Level"
              options={LEVEL_OPTIONS}
              selected={filters.levels}
              onApply={(levels) => onApplyFilters({ ...filters, levels })}
              onClose={close}
            />
          </FilterPopover>
        </div>

        <div className="lv-vheader-cell lv-col-service lv-vheader-filterable">
          <button
            className={`lv-vheader-btn ${filters.files.length ? 'lv-vheader-btn-active' : ''}`}
            onClick={() => setOpenFilter(openFilter === 'service' ? null : 'service')}
          >
            File
          </button>
          <FilterPopover open={openFilter === 'service'} onClose={close}>
            <CheckboxListFilter
              title="File"
              options={facets.files}
              selected={filters.files}
              onApply={(files) => onApplyFilters({ ...filters, files })}
              onClose={close}
              formatOption={stripExtension}
            />
          </FilterPopover>
        </div>

        <div className="lv-vheader-cell lv-col-event lv-vheader-filterable">
          <button
            className={`lv-vheader-btn ${filters.eventTypes.length ? 'lv-vheader-btn-active' : ''}`}
            onClick={() => setOpenFilter(openFilter === 'eventType' ? null : 'eventType')}
          >
            Event
          </button>
          <FilterPopover open={openFilter === 'eventType'} onClose={close}>
            <CheckboxListFilter
              title="Event Type"
              options={EVENT_TYPE_OPTIONS}
              selected={filters.eventTypes}
              onApply={(eventTypes) => onApplyFilters({ ...filters, eventTypes })}
              onClose={close}
            />
          </FilterPopover>
        </div>

        <div className="lv-vheader-cell lv-col-message lv-vheader-filterable">
          <button
            className={`lv-vheader-btn ${filters.messageContains ? 'lv-vheader-btn-active' : ''}`}
            onClick={() => setOpenFilter(openFilter === 'message' ? null : 'message')}
          >
            Message
          </button>
          <FilterPopover open={openFilter === 'message'} onClose={close}>
            <TextContainsFilter
              value={filters.messageContains}
              onApply={(messageContains) => onApplyFilters({ ...filters, messageContains })}
              onClose={close}
            />
          </FilterPopover>
        </div>

        <div className="lv-vheader-cell lv-col-source lv-vheader-filterable">
          <button
            className={`lv-vheader-btn ${filters.consoles.length || filters.scope !== 'all' ? 'lv-vheader-btn-active' : ''}`}
            onClick={() => setOpenFilter(openFilter === 'source' ? null : 'source')}
          >
            Source
          </button>
          <FilterPopover open={openFilter === 'source'} onClose={close}>
            <SourceFilter
              availableConsoles={facets.consoles}
              value={{ consoles: filters.consoles, scope: filters.scope }}
              onApply={({ consoles, scope }) => onApplyFilters({ ...filters, consoles, scope })}
              onClose={close}
            />
          </FilterPopover>
        </div>
      </div>

      {logs.length === 0 ? (
        hasActiveFilters ? (
          <div className="lv-empty lv-empty-inline">
            No logs match the current filters.
            <button className="lv-btn lv-btn-primary lv-empty-clear-btn" onClick={onClearFilters}>
              Clear all filters
            </button>
          </div>
        ) : (
          <div className="lv-empty lv-empty-inline">
            No logs yet. Drag a log folder into the area above to get started.
          </div>
        )
      ) : (
        <List
          rowComponent={LogRow}
          rowCount={logs.length}
          rowHeight={44}
          rowProps={{ logs }}
          className="lv-vlist"
          onRowsRendered={(visibleRows) => {
            if (visibleRows.stopIndex >= logs.length - FETCH_THRESHOLD) {
              onNearEnd();
            }
          }}
        />
      )}
    </div>
  );
};

export default LogTableReactWindow;

index.ts:
import express from 'express';
import cors from 'cors';
import multer from 'multer';
import fs from 'fs';
import path from 'path';
import { DuckDBInstance } from '@duckdb/node-api';

// BigInt -> JSON count(*) returns BigInt, JSON.stringify will crash without it.
(BigInt.prototype as any).toJSON = function () {
  return Number(this);
};

// SQL string literal escape: tek tırnağı ikile (' -> ''). Dosya adı/path'te tırnak olursa sorgu kırılmasın.
const sqlEscape = (s: string) => s.replace(/'/g, "''");

const app = express();
app.use(cors());
app.use(express.json());

const instance = await DuckDBInstance.create('logs.db');
const connection = await instance.connect();

await connection.run(`
  CREATE TABLE IF NOT EXISTS logs (
    timestamp VARCHAR,
    timestamp_iso TIMESTAMP,
    level VARCHAR,
    service VARCHAR,
    event_type VARCHAR,
    message VARCHAR,
    source_path VARCHAR,
    console INTEGER,
    is_service_log INTEGER,
    file_name VARCHAR
  )
`);

try {
  await connection.run(`ALTER TABLE logs ADD COLUMN file_name VARCHAR`);
} catch {
}

await connection.run(`
  UPDATE logs
  SET file_name = regexp_extract(source_path, '([^/]+)$', 1)
  WHERE file_name IS NULL AND source_path IS NOT NULL
`);

// multer: it writes files temporarly to the /uploads file
const uploadDir = path.join(process.cwd(), 'uploads');
if (!fs.existsSync(uploadDir)) {
  fs.mkdirSync(uploadDir);
}

const storage = multer.diskStorage({
  destination: (_req, _file, cb) => cb(null, uploadDir),
  filename: (_req, file, cb) => cb(null, `${Date.now()}-${file.originalname}`),
});

const upload = multer({ storage, limits: { fileSize: Infinity } });

// Windows'ta bir dosya hâlâ açık bir handle'a sahipse (AV taraması, native binary'nin
// handle'ı geç bırakması gibi) unlink EBUSY/EPERM ile başarısız olabilir — macOS/Linux'ta
// bu sorun yaşanmaz çünkü POSIX açık dosyaların silinmesine izin verir. Kısa gecikmeyle
// birkaç kez tekrar deniyoruz; hâlâ olmuyorsa gerçek hata kodunu logluyoruz.
// TODO: burayı staj bilgisayarında test edeceğim. çünkü macoste doğru şekilde çalışıyor ve /uploads klasörünü boşaltıyor.
async function safeUnlink(filePath: string, attempt = 1): Promise<void> {
  try {
    if (fs.existsSync(filePath)) {
      fs.unlinkSync(filePath);
    }
  } catch (err) {
    const code = (err as NodeJS.ErrnoException).code;
    console.error(
      `Temp file deletion failed (attempt ${attempt}): ${filePath} — code: ${code}`,
    );
    if (attempt < 3) {
      await new Promise((resolve) => setTimeout(resolve, 300 * attempt));
      return safeUnlink(filePath, attempt + 1);
    }
    console.error(`Giving up on deleting: ${filePath}`);
  }
}

/**
 * ENDPOINT: POST /upload
 * Uppy sends XHRUpload files with field of 'files', sendsr elativePath with meta field.
 * bundle: false + limit:1 -> each request brings 1 file -> req.body.relativePath belongs to that file.
 */
app.post('/upload', upload.array('files'), async (req, res) => {
  try {
    const files = req.files as Express.Multer.File[];
    if (!files || files.length === 0) {
      res.status(400).json({ error: 'There are no files to be uploaded.' });
      return;
    }
    console.log(`Files received: ${files.length}. Transfer to DuckDB started.`);

    for (const file of files) {
      // filePath: okumak için gerçek disk yolu (uploads/...)
      const filePath = sqlEscape(file.path.replace(/\\/g, '/'));

      // relPath: METADATA için mantıksal yol. Klasör yapısı (console_x, serviceLogs) burada.
      //! multer'ın temp yolunda bu bilgi yok; o yüzden Uppy'nin relativePath meta'sını kullanıyoruz.
      const relPath = sqlEscape(
        String(req.body.relativePath || file.originalname).replace(/\\/g, '/'),
      );

      try {
        await connection.run(`
        INSERT INTO logs
        SELECT
          timestamp,
          COALESCE(try_strptime(rebuilt_ts, '%d.%m.%Y %H:%M:%S.%f'), NOW()) AS timestamp_iso,
          level, service, event_type, message, source_path, console, is_service_log, file_name
        FROM (
          SELECT
            CAST(timestamp AS VARCHAR) AS timestamp,
            split_part(CAST(timestamp AS VARCHAR), ':', 1) || ':' ||
            split_part(CAST(timestamp AS VARCHAR), ':', 2) || ':' ||
            substr(rpad(split_part(CAST(timestamp AS VARCHAR), ':', 3), 5, '0'), 1, 2) || '.' ||
            substr(rpad(split_part(CAST(timestamp AS VARCHAR), ':', 3), 5, '0'), 3, 3) AS rebuilt_ts,
            CAST(level AS VARCHAR)     AS level,
            CAST(service AS VARCHAR)   AS service,
            CAST(eventType AS VARCHAR) AS event_type,
            CAST(message AS VARCHAR)   AS message,
            '${relPath}' AS source_path,
            TRY_CAST(regexp_extract('${relPath}', 'console[_-]?([0-9]+)', 1) AS INTEGER) AS console,
            CASE WHEN '${relPath}' LIKE '%serviceLogs%' THEN 1 ELSE 0 END AS is_service_log,
            regexp_extract('${relPath}', '([^/]+)$', 1) AS file_name
          FROM read_json_auto('${filePath}', format = 'newline_delimited',
                              ignore_errors = true, union_by_name = true)
        )
      `);
      } finally {
        await safeUnlink(filePath);
      }
    }

    const reader = await connection.runAndReadAll(
      'SELECT count(*) AS total FROM logs',
    );
    const totalRows = Number(reader.getRowObjects()[0]?.total || 0);

    console.log(`Done. Total rows in DB: ${totalRows}`);
    res.json({ success: true, totalInDB: totalRows });
  } catch (error) {
    console.error('Upload failed: ', (error as Error).message);
    res.status(500).json({ error: (error as Error).message });
  }
});

/**
 * ENDPOINT: GET /logs?page=&limit=
 * timestamp_iso DESC
 */
app.get('/logs', async (req, res) => {
  try {
    const page = Math.max(1, Number(req.query.page || 1));
    const limit = Math.max(1, Number(req.query.limit || 100));
    const offset = (page - 1) * limit;

    const levels = req.query.levels
      ? String(req.query.levels).split(',').filter(Boolean)
      : null;
    const eventTypes = req.query.eventTypes
      ? String(req.query.eventTypes).split(',').filter(Boolean)
      : null;
    const consoles = req.query.consoles
      ? String(req.query.consoles)
          .split(',')
          .map(Number)
          .filter((n) => !Number.isNaN(n))
      : null;
    const scope = req.query.scope as string | undefined; // 'service' | 'other' | undefined(=all)
    const from = req.query.from as string | undefined;
    const to = req.query.to as string | undefined;

    const conditions: string[] = [];
    const params: unknown[] = [];
    let i = 1; // $ placeholder counter

    if (levels?.length) {
      conditions.push(`level IN (${levels.map(() => `$${i++}`).join(', ')})`);
      params.push(...levels);
    }
    if (eventTypes?.length) {
      conditions.push(
        `event_type IN (${eventTypes.map(() => `$${i++}`).join(', ')})`,
      );
      params.push(...eventTypes);
    }
    const files = req.query.files
      ? String(req.query.files)
        .split(',')
        .filter(Boolean)
      : null;
    if (files?.length) {
      conditions.push(`file_name IN (${files.map(() => `$${i++}`).join(', ')})`);
      params.push(...files);
    }
    if (consoles?.length) {
      conditions.push(
        `console IN (${consoles.map(() => `$${i++}`).join(', ')})`,
      );
      params.push(...consoles);
    }
    if (scope === 'service') conditions.push('is_service_log = 1');
    if (scope === 'other') conditions.push('is_service_log = 0');
    if (from) {
      conditions.push(`timestamp_iso >= CAST($${i++} AS TIMESTAMP)`);
      params.push(from);
    }
    if (to) {
      conditions.push(`timestamp_iso <= CAST($${i++} AS TIMESTAMP)`);
      params.push(to);
    }

    if (req.query.messageContains) {
      conditions.push(`message ILIKE $${i++}`);
      params.push(`%${req.query.messageContains}%`);
    }

    const whereClause = conditions.length
      ? `WHERE ${conditions.join(' AND ')}`
      : '';

    const sortDir = req.query.sortDir === 'asc' ? 'ASC' : 'DESC';

    const sql = `
      SELECT * FROM logs
      ${whereClause}
      ORDER BY timestamp_iso ${sortDir}
      LIMIT $${i++} OFFSET $${i++}
    `;
    params.push(limit, offset);

    const prepared = await connection.prepare(sql);
    await prepared.bind(params);
    const reader = await prepared.runAndReadAll();

    res.json(reader.getRowObjects());
  } catch (error) {
    console.error('Query error: ', error);
    res.status(500).json({ error: (error as Error).message });
  }
});

/**
 * ENDPOINT: POST /reset
 */
app.post('/reset', async (_req, res) => {
  try {
    await connection.run('TRUNCATE TABLE logs');
    res.json({ message: 'Database successfully cleaned.' });
  } catch (error) {
    res.status(500).json({ error: (error as Error).message });
  }
});

/**
 * ENDPOINT: GET /facets
 * 
 * Filtre popover'larının seçenek listelerini doldurur. Service ve console açık
 * uçlu (kaç tane olduğu baştan bilinmiyor) olduğu için distinct değerleri
 * veritabanından çekiyoruz. Level/eventType zaten sabit bir kelime hazinesine
 * sahip, onlar için sorguya gerek yok (frontend'de sabit dizi).
 */
app.get('/facets', async (_req, res) => {
  try {
    const filesReader = await connection.runAndReadAll(
      'SELECT DISTINCT file_name FROM logs WHERE file_name IS NOT NULL ORDER BY file_name'
    );
    const consolesReader = await connection.runAndReadAll(
      'SELECT DISTINCT console FROM logs WHERE console IS NOT NULL ORDER BY console'
    );

    res.json({
      files: filesReader.getRowObjects().map((r) => r.file_name as string),
      consoles: consolesReader.getRowObjects().map((r) => r.console as number),
    });
  } catch (error) {
    console.error('Facets query error: ', error);
    res.status(500).json({ error: (error as Error).message });
  }
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Log server is ready: http://localhost:${PORT}`);
});

logviewer.css:
html,
body,
#root {
  width: 100%;
  min-height: 100vh;
}
#root {
  max-width: none;
  margin: 0;
  padding: 0;
  text-align: left;
  display: block;
}

:root {
  --lv-bg: #0b0e14;
  --lv-surface: #121822;
  --lv-surface-2: #161d29;
  --lv-border: #232c3b;
  --lv-text: #d6dee8;
  --lv-muted: #6b7787;
  --lv-accent: #4fd1c5;
  --lv-error: #f76d6d;
  --lv-warn: #e3b341;
  --lv-info: #5aa2f0;
  --lv-mono:
    ui-monospace, 'SF Mono', SFMono-Regular, Menlo, Consolas, monospace;
  --lv-sans: system-ui, -apple-system, 'Segoe UI', Roboto, sans-serif;
}
* {
  box-sizing: border-box;
}
body {
  margin: 0;
  background: var(--lv-bg);
  color: var(--lv-text);
  font-family: var(--lv-sans);
}

.lv-shell {
  max-width: 1400px;
  margin: 0 auto;
  padding: 28px 24px 64px;
}

.lv-header {
  display: flex;
  align-items: center;
  gap: 16px;
  margin-bottom: 24px;
}
.lv-brand {
  display: flex;
  flex-direction: column;
  gap: 2px;
}
.lv-brand h1 {
  margin: 0;
  font-size: 20px;
  font-weight: 650;
  letter-spacing: -0.01em;
}
.lv-brand span {
  font-family: var(--lv-mono);
  font-size: 12px;
  color: var(--lv-muted);
}
.lv-spacer {
  flex: 1;
}

.lv-btn {
  font-family: var(--lv-sans);
  font-size: 13px;
  font-weight: 550;
  padding: 9px 14px;
  border-radius: 8px;
  cursor: pointer;
  border: 1px solid var(--lv-border);
  background: var(--lv-surface);
  color: var(--lv-text);
  transition:
    background 0.15s,
    border-color 0.15s,
    opacity 0.15s;
}
.lv-btn:hover:not(:disabled) {
  background: var(--lv-surface-2);
  border-color: #2f3a4c;
}
.lv-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.lv-btn-danger {
  color: var(--lv-error);
  border-color: rgba(247, 109, 109, 0.35);
  background: rgba(247, 109, 109, 0.08);
}
.lv-btn-danger:hover {
  background: rgba(247, 109, 109, 0.16);
}

.lv-toolbar {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 20px 0 14px;
}
.lv-page {
  font-family: var(--lv-mono);
  font-size: 13px;
  color: var(--lv-muted);
  min-width: 92px;
  text-align: center;
}
.lv-loading {
  font-family: var(--lv-mono);
  font-size: 12px;
  color: var(--lv-accent);
}

/* ── Table ── */
.lv-tablewrap {
  border: 1px solid var(--lv-border);
  border-radius: 12px;
  overflow: auto;
  max-height: 65vh;
  background: var(--lv-surface);
}
.lv-tablewrap::-webkit-scrollbar {
  width: 10px;
  height: 10px;
}
.lv-tablewrap::-webkit-scrollbar-thumb {
  background: #2a3445;
  border-radius: 8px;
  border: 2px solid var(--lv-surface);
}
.lv-tablewrap::-webkit-scrollbar-track {
  background: transparent;
}

.lv-table {
  width: 100%;
  border-collapse: collapse;
  font-family: var(--lv-mono);
  font-size: 12.5px;
}
.lv-table thead th {
  position: sticky;
  top: 0;
  z-index: 1;
  background: var(--lv-surface-2);
  color: var(--lv-muted);
  font-family: var(--lv-sans);
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  text-align: left;
  padding: 11px 14px;
  border-bottom: 1px solid var(--lv-border);
  white-space: nowrap;
}
.lv-table tbody tr {
  border-bottom: 1px solid rgba(35, 44, 59, 0.55);
}
.lv-table tbody tr:hover {
  background: var(--lv-surface-2);
}
.lv-table td {
  padding: 8px 14px;
  vertical-align: middle;
}

.lv-td-stripe {
  border-left: 3px solid transparent;
}
.lv-stripe-error {
  border-left-color: var(--lv-error);
}
.lv-stripe-warning {
  border-left-color: var(--lv-warn);
}
.lv-stripe-info {
  border-left-color: var(--lv-info);
}

.lv-ts-date {
  color: var(--lv-muted);
}
.lv-ts-time {
  color: var(--lv-text);
}
.lv-service {
  color: var(--lv-accent);
}
.lv-event {
  color: var(--lv-muted);
}
.lv-msg {
  color: var(--lv-text);
  max-width: 520px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.lv-src {
  color: #4a5566;
  font-size: 11px;
  max-width: 220px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  direction: rtl;
  text-align: left;
}

.lv-badge {
  display: inline-flex;
  align-items: center;
  gap: 6px;
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.03em;
  padding: 3px 9px;
  border-radius: 999px;
  text-transform: uppercase;
}
.lv-badge::before {
  content: '';
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: currentColor;
}
.lv-badge-error {
  color: var(--lv-error);
  background: rgba(247, 109, 109, 0.12);
}
.lv-badge-warning {
  color: var(--lv-warn);
  background: rgba(227, 179, 65, 0.12);
}
.lv-badge-info {
  color: var(--lv-info);
  background: rgba(90, 162, 240, 0.12);
}

.lv-empty {
  border: 1px dashed var(--lv-border);
  border-radius: 12px;
  padding: 56px 20px;
  text-align: center;
  color: var(--lv-muted);
  font-size: 14px;
  background: var(--lv-surface);
}

.lv-vrow:hover {
  background: var(--lv-surface-2);
}

/* Virtualized table (react-window) */
.lv-vheader {
  display: flex;
  position: sticky;
  top: 0;
  z-index: 2;
  background: var(--lv-surface-2);
  border-bottom: 1px solid var(--lv-border);
  font-family: var(--lv-sans);
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--lv-muted);
}
.lv-vheader-cell {
  padding: 11px 14px;
}

.lv-vlist {
  width: 100%;
  height: calc(65vh - 40px);
  overflow-y: auto;
  overflow-x: hidden;
}

.lv-vrow {
  display: flex;
  align-items: center;
  border-bottom: 1px solid rgba(35, 44, 59, 0.55);
  box-sizing: border-box;
}
.lv-vrow:hover {
  background: var(--lv-surface-2);
}

.lv-vcell {
  padding: 8px 14px;
  box-sizing: border-box;
}

.lv-col-timestamp {
  width: 180px;
  flex-shrink: 0;
}
.lv-col-level {
  width: 110px;
  flex-shrink: 0;
}
.lv-col-service {
  width: 150px;
  flex-shrink: 0;
}
.lv-col-event {
  width: 150px;
  flex-shrink: 0;
}

.lv-cell-clip {
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.lv-col-message {
  flex-grow: 1;
  min-width: 300px;
}
.lv-col-source {
  width: 220px;
  flex-shrink: 0;
}

.lv-vheader-filterable {
  position: relative;
  padding: 0;
}
.lv-vheader-btn {
  all: unset;
  display: flex;
  align-items: center;
  width: 100%;
  height: 100%;
  padding: 11px 14px;
  cursor: pointer;
  box-sizing: border-box;
}
.lv-vheader-btn:hover {
  background: rgba(255, 255, 255, 0.04);
}
.lv-vheader-btn-active {
  color: var(--lv-accent);
}
.lv-vheader-btn-active::after {
  content: '';
  position: absolute;
  top: 8px;
  right: 6px;
  width: 6px;
  height: 6px;
  border-radius: 50%;
  background: var(--lv-accent);
}

.lv-popover {
  position: absolute;
  top: calc(100% + 4px);
  left: 0;
  z-index: 10;
  min-width: 220px;
  max-width: 320px; /* 280 → 320 */
  background: var(--lv-surface);
  border: 1px solid var(--lv-border);
  border-radius: 10px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.4);
  padding: 12px;
}
.lv-popover-body {
  display: flex;
  flex-direction: column;
  gap: 10px;
}
.lv-popover-title {
  font-family: var(--lv-sans);
  font-size: 11px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: var(--lv-muted);
}
.lv-popover-list {
  display: flex;
  flex-direction: column;
  gap: 6px;
  max-height: 180px;
  overflow-y: auto;
}
.lv-checkbox-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-family: var(--lv-mono);
  font-size: 12.5px;
  color: var(--lv-text);
  cursor: pointer;
}
.lv-popover-field {
  display: flex;
  flex-direction: column;
  gap: 4px;
  font-family: var(--lv-sans);
  font-size: 11px;
  color: var(--lv-muted);
}
.lv-popover-input {
  font-family: var(--lv-mono);
  font-size: 12.5px;
  color: var(--lv-text);
  background: var(--lv-surface-2);
  border: 1px solid var(--lv-border);
  border-radius: 6px;
  padding: 6px 8px;
}
.lv-popover-actions {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
  margin-top: 4px;
}
.lv-btn-primary {
  background: var(--lv-accent);
  border-color: var(--lv-accent);
  color: #0b0e14;
}
.lv-btn-primary:hover {
  opacity: 0.9;
}

.lv-vheader-inner {
  display: flex;
  align-items: stretch;
  width: 100%;
  height: 100%;
}
.lv-vheader-inner .lv-vheader-btn {
  flex: 1;
  width: auto;
}

.lv-sort-btn {
  all: unset;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 28px;
  cursor: pointer;
  color: var(--lv-muted);
  font-size: 10px;
  box-sizing: border-box;
}
.lv-sort-btn:hover {
  color: var(--lv-accent);
}

.lv-popover-input {
  color-scheme: dark;
  width: 100%;
  box-sizing: border-box;
}

.lv-popover-error {
  font-family: var(--lv-sans);
  font-size: 11px;
  color: var(--lv-error);
}

.lv-btn-primary:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}

.lv-time-select-row {
  display: flex;
  align-items: center;
  gap: 4px;
  margin-top: 4px;
}
.lv-popover-select {
  flex: 1;
  font-family: var(--lv-mono);
  font-size: 12.5px;
  color: var(--lv-text);
  background: var(--lv-surface-2);
  border: 1px solid var(--lv-border);
  border-radius: 6px;
  padding: 6px 8px;
  cursor: pointer;
}
.lv-time-colon {
  color: var(--lv-muted);
  font-family: var(--lv-mono);
  font-size: 13px;
}

.lv-date-triple { display: flex; align-items: center; gap: 4px; margin-top: 4px; }
.lv-date-part { width: 40px; text-align: center; flex: none; }
.lv-date-part-year { width: 56px; }
.lv-date-sep { color: var(--lv-muted); font-family: var(--lv-mono); font-size: 13px; }

.lv-empty-inline { border: none; border-radius: 0; }
.lv-empty-clear-btn { display: block; margin: 12px auto 0; }

/* react-datepicker — koyu tema */
.react-datepicker-popper { z-index: 50; }

.react-datepicker {
  font-family: var(--lv-sans);
  background: var(--lv-surface);
  border: 1px solid var(--lv-border);
  border-radius: 10px;
  overflow: hidden;
}
.react-datepicker__header {
  background: var(--lv-surface-2);
  border-bottom: 1px solid var(--lv-border);
}
.react-datepicker__current-month,
.react-datepicker__day-name,
.react-datepicker-time__header {
  color: var(--lv-text);
}
.react-datepicker__day { color: var(--lv-text); }
.react-datepicker__day:hover { background: var(--lv-surface-2); }
.react-datepicker__day--selected,
.react-datepicker__day--keyboard-selected {
  background: var(--lv-accent);
  color: #0B0E14;
}
.react-datepicker__day--outside-month { color: var(--lv-muted); }
.react-datepicker__navigation-icon::before { border-color: var(--lv-muted); }
.react-datepicker__triangle { display: none; }

.react-datepicker__time-container { border-left: 1px solid var(--lv-border); }
.react-datepicker__time,
.react-datepicker__time-box { background: var(--lv-surface); }
.react-datepicker__time-list-item { color: var(--lv-text); }
.react-datepicker__time-list-item:hover { background: var(--lv-surface-2) !important; }
.react-datepicker__time-list-item--selected {
  background: var(--lv-accent) !important;
  color: #0B0E14 !important;
}

.react-datepicker__input-container input {
  color-scheme: dark;
  width: 100%;
  box-sizing: border-box;
}

#lv-datepicker-portal { position: relative; z-index: 100; }

FilterPopover.tsx:
import { useEffect, useRef, type ReactNode } from 'react';

interface FilterPopoverProps {
  open: boolean;
  onClose: () => void;
  children: ReactNode;
}

const FilterPopover = ({ open, onClose, children }: FilterPopoverProps) => {
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    if (!open) return;

    const handleClickOutside = (event: MouseEvent) => {
      const target = event.target as Node;

      const clickedInsidePopover = ref.current?.contains(target) ?? false;
      
      const portalEl = document.getElementById('lv-datepicker-portal');
      const clickedInsidePortal = portalEl?.contains(target) ?? false;

      if (!clickedInsidePopover && !clickedInsidePortal) {
        onClose();
      }
    };

    document.addEventListener('mousedown', handleClickOutside);
    return () => document.removeEventListener('mousedown', handleClickOutside);
  }, [open, onClose]);

  if (!open) return null;

  return (
    <div className="lv-popover" ref={ref}>
      {children}
    </div>
  );
};

export default FilterPopover;

FilterContent.tsx:
import { useState } from 'react';
import DatePicker from 'react-datepicker';
import { format, isAfter } from 'date-fns';
import 'react-datepicker/dist/react-datepicker.css';

interface CheckboxListFilterProps {
  title: string;
  options: string[];
  selected: string[];
  onApply: (selected: string[]) => void;
  onClose: () => void;
  formatOption?: (opt: string) => string;
}

export const CheckboxListFilter = ({ title, options, selected, onApply, onClose, formatOption }: CheckboxListFilterProps) => {
  const [draft, setDraft] = useState<string[]>(selected);

  const toggle = (opt: string) => {
    setDraft((prev) => (prev.includes(opt) ? prev.filter((o) => o !== opt) : [...prev, opt]));
  };

  return (
    <div className="lv-popover-body">
      <div className="lv-popover-title">{title}</div>
      <div className="lv-popover-list">
        {options.map((opt) => (
          <label key={opt} className="lv-checkbox-row">
            <input type="checkbox" checked={draft.includes(opt)} onChange={() => toggle(opt)} />
            <span>{formatOption ? formatOption(opt) : opt}</span>
          </label>
        ))}
      </div>
      <div className="lv-popover-actions">
        <button className="lv-btn" onClick={() => setDraft([])}>Clear</button>
        <button className="lv-btn lv-btn-primary" onClick={() => { onApply(draft); onClose(); }}>Apply</button>
      </div>
    </div>
  );
};

interface SourceFilterValue {
  consoles: number[];
  scope: 'all' | 'service' | 'other';
}

interface SourceFilterProps {
  availableConsoles: number[];
  value: SourceFilterValue;
  onApply: (value: SourceFilterValue) => void;
  onClose: () => void;
}

export const SourceFilter = ({ availableConsoles, value, onApply, onClose }: SourceFilterProps) => {
  const [draftConsoles, setDraftConsoles] = useState<number[]>(value.consoles);
  const [draftScope, setDraftScope] = useState<SourceFilterValue['scope']>(value.scope);

  const toggleConsole = (c: number) => {
    setDraftConsoles((prev) => (prev.includes(c) ? prev.filter((x) => x !== c) : [...prev, c]));
  };

  return (
    <div className="lv-popover-body">
      <div className="lv-popover-title">Console</div>
      <div className="lv-popover-list">
        {availableConsoles.map((c) => (
          <label key={c} className="lv-checkbox-row">
            <input type="checkbox" checked={draftConsoles.includes(c)} onChange={() => toggleConsole(c)} />
            <span>Console {c}</span>
          </label>
        ))}
      </div>

      <div className="lv-popover-title">Scope</div>
      <div className="lv-popover-list">
        {(['all', 'service', 'other'] as const).map((s) => (
          <label key={s} className="lv-checkbox-row">
            <input type="radio" name="scope" checked={draftScope === s} onChange={() => setDraftScope(s)} />
            <span>{s === 'all' ? 'Show all logs' : s === 'service' ? 'Only service logs' : 'Only other logs'}</span>
          </label>
        ))}
      </div>

      <div className="lv-popover-actions">
        <button className="lv-btn" onClick={() => { setDraftConsoles([]); setDraftScope('all'); }}>Clear</button>
        <button
          className="lv-btn lv-btn-primary"
          onClick={() => { onApply({ consoles: draftConsoles, scope: draftScope }); onClose(); }}
        >
          Apply
        </button>
      </div>
    </div>
  );
};

interface RangeFilterValue {
  from: string | null;
  to: string | null;
}

interface RangeFilterProps {
  value: RangeFilterValue;
  onApply: (value: RangeFilterValue) => void;
  onClose: () => void;
}

// Backend "YYYY-MM-DDTHH:mm" string bekliyor -> DatePicker'ın çalıştığı Date objesine çevir
function parseIso(value: string | null): Date | null {
  if (!value) return null;
  const d = new Date(value);
  return Number.isNaN(d.getTime()) ? null : d;
}

export const RangeFilter = ({ value, onApply, onClose }: RangeFilterProps) => {
  const [fromDate, setFromDate] = useState<Date | null>(parseIso(value.from));
  const [toDate, setToDate] = useState<Date | null>(parseIso(value.to));

  const isInvalidRange = fromDate && toDate && isAfter(fromDate, toDate);

  return (
    <div className="lv-popover-body">
      <div className="lv-popover-title">Timestamp range</div>

      <div className="lv-popover-field">
        <span>From</span>
        <DatePicker
          selected={fromDate}
          onChange={(date) => setFromDate(date)}
          showTimeSelect
          timeIntervals={5}
          locale="tr"
          dateFormat="dd.MM.yyyy HH:mm"
          placeholderText="gg.aa.yyyy ss:dd"
          className="lv-popover-input"
          isClearable
          portalId="lv-datepicker-portal"
        />
      </div>

      <div className="lv-popover-field">
        <span>To</span>
        <DatePicker
          selected={toDate}
          onChange={(date) => setToDate(date)}
          showTimeSelect
          timeIntervals={5}
          locale="tr"
          dateFormat="dd.MM.yyyy HH:mm"
          placeholderText="gg.aa.yyyy ss:dd"
          className="lv-popover-input"
          isClearable
          portalId="lv-datepicker-portal"
        />
      </div>

      {isInvalidRange && <div className="lv-popover-error">"From" must be before "To"</div>}
      <div className="lv-popover-actions">
        <button className="lv-btn" onClick={() => { setFromDate(null); setToDate(null); }}>Clear</button>
        <button
          className="lv-btn lv-btn-primary"
          disabled={!!isInvalidRange}
          onClick={() => {
            onApply({
              from: fromDate ? format(fromDate, "yyyy-MM-dd'T'HH:mm") : null,
              to: toDate ? format(toDate, "yyyy-MM-dd'T'HH:mm") : null,
            });
            onClose();
          }}
        >
          Apply
        </button>
      </div>
    </div>
  );
};

interface TextContainsFilterProps {
  value: string;
  onApply: (value: string) => void;
  onClose: () => void;
}

export const TextContainsFilter = ({ value, onApply, onClose }: TextContainsFilterProps) => {
  const [draft, setDraft] = useState(value);

  return (
    <div className="lv-popover-body">
      <div className="lv-popover-title">Message contains</div>
      <input
        type="text"
        className="lv-popover-input"
        placeholder="e.g. timeout, .., error code"
        value={draft}
        onChange={(e) => setDraft(e.target.value)}
        maxLength={200}
        autoFocus
      />
      <div className="lv-popover-actions">
        <button className="lv-btn" onClick={() => setDraft('')}>Clear</button>
        <button className="lv-btn lv-btn-primary" onClick={() => { onApply(draft); onClose(); }}>Apply</button>
      </div>
    </div>
  );
};


