# Media Service

## Service Responsibility

The media service owns practice sessions, video upload flow, signed URLs, storage metadata, and links between videos and athletes.

## Video Upload Flow

1. Frontend requests signed upload URL.
2. Media service creates pending video record.
3. Frontend uploads directly to storage.
4. Frontend confirms upload completion.
5. Media service marks video as uploaded and links it to a session.

## Signed URL Flow

The backend generates time-limited upload URLs so video files do not pass through the application server.

## Storage Provider

Use cloud object storage for production. Local development may use a mock provider, MinIO, or local metadata-only flow.

## Video Metadata

Store file name, storage key, content type, size, duration, upload status, athlete ID, coach ID, and practice session ID.

## Practice Sessions

Practice sessions group videos and notes by athlete, coach, date, and session type.

## Endpoints

- `POST /practice-sessions`
- `GET /practice-sessions/{session_id}`
- `POST /videos/upload-url`
- `POST /videos/{video_id}/complete-upload`
- `GET /videos/{video_id}`
- `GET /athletes/{athlete_id}/videos`

## Tables Owned

- `practice_sessions`
- `videos`
- Future: `video_transcripts`, `video_tags`

## Testing Plan

- Signed URL creation
- Pending video record creation
- Upload completion
- Athlete video listing
- Access control by coach

## Future Improvements

- Transcoding pipeline
- Thumbnail generation
- Video annotations
- Transcript extraction
- Background upload processing
# Timeline Outbox

Practice lifecycle, first verified upload completion, and video deletion insert timeline payloads in the same transaction as domain state. `python -m app.workers.outbox_publisher` delivers them. `python -m app.workers.outbox_admin inspect` reports queue state and `retry-failed` safely requeues failed rows.
