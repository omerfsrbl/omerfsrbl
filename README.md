App.tsx:
import { useEffect, useState } from 'react';
import FileUploader from './Components/FileUploader';
import LogTable from './Components/LogTable';
import LogTableReactWindow from './Components/LogTableReactWindow';

import './styles/logviewer.css';

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

function App() {
  const [logs, setLogs] = useState<LogItem[]>([]);
  const [page, setPage] = useState(1);
  const [loading, setLoading] = useState(false);

  const fetchLogs = async (targetPage: number) => {
    setLoading(true);
    try {
      const response = await fetch(
        // ! limit can be change. limit changes here.
        `http://localhost:3000/logs?page=${targetPage}&limit=10000000`,
      );
      const data = await response.json();
      if (Array.isArray(data)) {
        setLogs(data);
      } else {
        console.error('Response is not an array:', data);
        setLogs([]);
      }
      setPage(targetPage);
    } catch (error) {
      console.error('Fetch failed:', error);
    } finally {
      setLoading(false);
    }
  };

  const onClickResetDatabase = async () => {
    try {
      await fetch('http://localhost:3000/reset', { method: 'POST' });
      setLogs([]);
      setPage(1);
    } catch (error) {
      console.error('Reset failed:', error);
    }
  };

  useEffect(() => {
    fetchLogs(1);
  }, []);

  return (
    <>
      <div className="lv-shell">
        <header className="lv-header">
          <div className="lv-brand">
            <h1>Log Viewer</h1>
            <span>console inspector</span>
          </div>
          <div className="lv-spacer" />
          <button
            className="lv-btn lv-btn-danger"
            onClick={onClickResetDatabase}
          >
            Clear logs
          </button>
        </header>

        <FileUploader onUploadComplete={() => fetchLogs(1)} />

        <div className="lv-toolbar">
          <button
            className="lv-btn"
            disabled={page === 1 || loading}
            onClick={() => fetchLogs(page - 1)}
          >
            Previous
          </button>
          <span className="lv-page">Page {page}</span>
          <button
            className="lv-btn"
            disabled={logs.length < 100 || loading}
            onClick={() => fetchLogs(page + 1)}
          >
            Next
          </button>
          {loading && <span className="lv-loading">loading…</span>}
        </div>

        {/* <LogTable logs={logs} /> */}
        <LogTableReactWindow logs={logs} />
      </div>
    </>
  );
}

export default App;

FileUploader.txs:
import { useEffect, useState } from 'react';
import { Uppy } from '@uppy/core';
import Dashboard from '@uppy/react/dashboard';
import XHRUpload from '@uppy/xhr-upload';

import '@uppy/core/css/style.min.css';
import '@uppy/dashboard/css/style.min.css';

// TODO: interface classlarda kullanılırken burada neden kullandık? sırf typescript hata vermesin diye mi? use case olarak type mı interface mi daha avantajlı, hangisi kullanılıyor?
interface FileUploaderProps {
  onUploadComplete: () => void;
}

const FileUploader = ({ onUploadComplete }: FileUploaderProps) => {
  const [uppy] = useState(() => {
    const uppyInstance = new Uppy({
      id: 'File-Uploader',
      restrictions: { allowedFileTypes: ['.json', '.ndjson'] },
      autoProceed: false,
    });

    uppyInstance.use(XHRUpload, {
      endpoint: 'http://localhost:3000/upload',
      fieldName: 'files',
      bundle: false,
      limit: 1,
      timeout: 0,
      headers: { accept: 'application/json' },
    });

    return uppyInstance;
  });

  useEffect(() => {
    const successHandler = (file: any, response: any) => {
      console.log('Uploaded:', file?.name, response);
    };
    const errorHandler = (file: any, response: any) => {
      console.log('Upload error:', file?.name, response);
    };
    // ! after the succesfull upload; file uploader tells that file uploaded successfully, you can continue to the app.tsx. after the function call, page refreshes and brand new :d logs comes to the page. 
    const completeHandler = (result: any) => {
      if (result.successful.length > 0) onUploadComplete();
    };

    uppy.on('upload-success', successHandler);
    uppy.on('upload-error', errorHandler);
    uppy.on('complete', completeHandler);

    return () => {
      uppy.off('upload-success', successHandler);
      uppy.off('upload-error', errorHandler);
      uppy.off('complete', completeHandler);
    };
  }, [uppy, onUploadComplete]);

  return (
    <Dashboard
      uppy={uppy}
      theme="dark"
      height={260}
      width="100%"
      note="Drag a log folder here · .json / .ndjson"
      proudlyDisplayPoweredByUppy={false}
      showLinkToFileUploadResult={false}
      showRemoveButtonAfterComplete={true}
    />
  );
};

export default FileUploader;

LogTable.tsx:
interface LogItem {
    timestamp: string;
    level: string;
    service: string;
    event_type: string;
    message: string;
    source_path: string;
    console: number | null;
    is_service_log: number;
  }
  
  interface LogTableProps {
    logs: LogItem[];
  }
  
  const levelKey = (level: string) => {
    const l = level.toLowerCase();
    if (l === 'error' || l === 'warning' || l === 'info') return l;
    return 'info';
  };
  
  const LogTable = ({ logs }: LogTableProps) => {
    if (logs.length === 0) {
      return (
        <div className="lv-empty">
          No logs yet. Drag a log folder into the area above to get started.
        </div>
      );
    }
  
    return (
      <div className="lv-tablewrap">
        <table className="lv-table">
          <thead>
            <tr>
              <th>Timestamp</th>
              <th>Level</th>
              <th>Service</th>
              <th>Event</th>
              <th>Message</th>
              <th>Source</th>
            </tr>
          </thead>
          <tbody>
            {logs.map((log, index) => {
              const lvl = levelKey(log.level);
// ! buranın direkt date, time olmamasının nedeni genel kullanım bu şekildeymiş. bu projede gereksiz ancak herhangi bir başka projede date time utc gibi 1den fazla boşluk olma duruunda patlamaması için ekstra bir güvenlik önlemiymiş.
              const [date, ...rest] = log.timestamp.split(' ');
              const time = rest.join(' ');
              return (
                <tr key={index}>
                  <td className={`lv-td-stripe lv-stripe-${lvl}`}>
                    <span className="lv-ts-date">{date} </span>
                    <span className="lv-ts-time">{time}</span>
                  </td>
                  <td>
                    <span className={`lv-badge lv-badge-${lvl}`}>
                      {log.level}
                    </span>
                  </td>
                  <td className="lv-service">{log.service}</td>
                  <td className="lv-event">{log.event_type}</td>
                  <td className="lv-msg" title={log.message}>
                    {log.message}
                  </td>
                  <td className="lv-src" title={log.source_path}>
                    {log.source_path}
                  </td>
                </tr>
              );
            })}
          </tbody>
        </table>
      </div>
    );
  };
  
  export default LogTable;


  LogTableReactWindow.txs:
  import { List, type RowComponentProps } from "react-window";
import React from 'react';

interface LogItem {
  timestamp: string;
  level: string;
  service: string;
  event_type: string;
  message: string;
  source_path: string;
  console: number | null;
  is_service_log: number;
}

interface LogTableProps {
  logs: LogItem[];
}

const levelKey = (level: string) => {
  const l = level.toLowerCase();
  if (l === 'error' || l === 'warning' || l === 'info') return l;
  return 'info';
};

// 1. Hücrelerin genişliklerini ve hizalamalarını sabitleyen inline stiller
// CSS'teki orijinal tablo sütun yapısını taklit etmek için flex paylaşımları
const cellStyles = {
  timestamp: { width: '180px', flexShrink: 0 },
  level:     { width: '110px', flexShrink: 0 },
  service:   { width: '150px', flexShrink: 0 },
  event:     { width: '150px', flexShrink: 0 },
  message:   { flexGrow: 1, minWidth: '300px' }, // max-width ve kırpma CSS'te zaten tanımlı (.lv-msg)
  source:    { width: '220px', flexShrink: 0 }  // max-width ve rtl CSS'te zaten tanımlı (.lv-src)
};

function LogRow({
  index,
  style,
  logs
}: RowComponentProps<{
  logs: LogItem[];
}>) {
  const log = logs[index];
  const lvl = levelKey(log.level);
  const [date, ...rest] = log.timestamp.split(' ');
  const time = rest.join(' ');

  // react-window'un getirdiği absolute koordinat stilleri ile 
  // bizim flex satır yapımızı (display: flex satır içi vererek) birleştiriyoruz.
  const combinedRowStyle = {
    ...style,
    display: 'flex',
    alignItems: 'center',
    borderBottom: '1px solid rgba(35, 44, 59, 0.55)', // CSS'teki tr border'ı
    boxSizing: 'border-box' as const
  };

  const combinedTdStyle = {
    padding: '8px 14px',
    display: 'inline-block',
    verticalAlign: 'middle',
    boxSizing: 'border-box' as const
  };

  return (
    // CSS'teki hover animasyonunu korumak için satıra ekstra bir sınıf yazmıyoruz, flex yapısını inline çözüyoruz.
    <div style={combinedRowStyle}>
      <div className={`lv-td-stripe lv-stripe-${lvl}`} style={{ ...combinedTdStyle, ...cellStyles.timestamp }}>
        <span className="lv-ts-date">{date} </span>
        <span className="lv-ts-time">{time}</span>
      </div>
      <div style={{ ...combinedTdStyle, ...cellStyles.level }}>
        <span className={`lv-badge lv-badge-${lvl}`}>
          {log.level}
        </span>
      </div>
      <div className="lv-service" style={{ ...combinedTdStyle, ...cellStyles.service }}>{log.service}</div>
      <div className="lv-event" style={{ ...combinedTdStyle, ...cellStyles.event }}>{log.event_type}</div>
      <div className="lv-msg" title={log.message} style={{ ...combinedTdStyle, ...cellStyles.message }}>
        {log.message}
      </div>
      <div className="lv-src" title={log.source_path} style={{ ...combinedTdStyle, ...cellStyles.source }}>
        {log.source_path}
      </div>
    </div>
  );
}

const LogTableReactWindow = ({ logs }: LogTableProps) => {
  if (logs.length === 0) {
    return (
      <div className="lv-empty">
        No logs yet. Drag a log folder into the area above to get started.
      </div>
    );
  }

  return (
    <div className="lv-tablewrap">
      <div style={{
        display: 'flex',
        position: 'sticky',
        top: 0,
        zIndex: 2,
        background: 'var(--lv-surface-2)',
        borderBottom: '1px solid var(--lv-border)',
        fontFamily: 'var(--lv-sans)',
        fontSize: '11px',
        fontWeight: 600,
        textTransform: 'uppercase',
        letterSpacing: '0.06em',
        color: 'var(--lv-muted)'
      }}>
        <div style={{ padding: '11px 14px', ...cellStyles.timestamp }}>Timestamp</div>
        <div style={{ padding: '11px 14px', ...cellStyles.level }}>Level</div>
        <div style={{ padding: '11px 14px', ...cellStyles.service }}>Service</div>
        <div style={{ padding: '11px 14px', ...cellStyles.event }}>Event</div>
        <div style={{ padding: '11px 14px', ...cellStyles.message }}>Message</div>
        <div style={{ padding: '11px 14px', ...cellStyles.source }}>Source</div>
      </div>

      <List
        rowComponent={LogRow}
        rowCount={logs.length}
        rowHeight={38} 
        rowProps={{ logs }}
        className="lv-table" 
        style={{ 
          height: 'calc(65vh - 40px)',
          width: '100%',
          overflowX: 'auto'
        }}
      />
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

// timestamp_iso: original timestamp -> iso form
// source_path / console / is_service_log: files location (comes from relativePath)
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
    is_service_log INTEGER
  )
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
            level, service, event_type, message, source_path, console, is_service_log
          FROM (
            SELECT
              CAST(timestamp AS VARCHAR) AS timestamp,
              -- saniye+ms kuyruğunu 5 haneye rpad'le, ilk 2 = saniye, son 3 = ms, "SS.mmm" formuna sok
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
              CASE WHEN '${relPath}' LIKE '%serviceLogs%' THEN 1 ELSE 0 END AS is_service_log
            FROM read_json_auto('${filePath}', format = 'newline_delimited',
                                ignore_errors = true, union_by_name = true)
          )
        `);
      } finally {
        // TODO: yükleme tamamlandıktan sonra temp file'ın içindekiler silinmedi buna bak. !!!!!!silinmesi lazım ?!?!?!?!!?!?!?!?
        // delete the temp file even if insert fails (uploads/ shouldn't get full)
        if (fs.existsSync(file.path)) fs.unlinkSync(file.path);
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

    const reader = await connection.runAndReadAll(`
      SELECT * FROM logs
      ORDER BY timestamp_iso DESC
      LIMIT ${limit} OFFSET ${offset}
    `);

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

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Log server is ready: http://localhost:${PORT}`);
});


