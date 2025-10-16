# การบันทึกและเฝ้าระวังระบบ (Logging & Monitoring)

## รูปแบบ Log
```json
{
  "level": "info",
  "message": "สร้างผู้ใช้สำเร็จ",
  "timestamp": "2025-10-14T07:00:00Z",
  "requestId": "abc-123"
}
```

## Error Log
- ต้องมี stack trace และ `requestId`
- ระดับ log: debug, info, warn, error, fatal

## ตัวชี้วัด (Metrics)
- latency (ms)
- error rate (%)
- throughput (req/sec)

## การติดตาม (Tracing)
- ใช้ `X-Trace-ID` ติดตามระหว่าง service
