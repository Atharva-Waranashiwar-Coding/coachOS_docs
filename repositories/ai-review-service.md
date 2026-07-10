# AI Review Service

## Service Responsibility

The AI review service owns AI-generated summaries, observations, recommendations, and the coach review approval workflow.

## AI Review Generation Flow

1. Coach requests review for a video.
2. Service loads video metadata and athlete context.
3. Service builds prompt with structured instructions.
4. AI provider returns structured review output.
5. Service stores draft review content.
6. Coach edits, approves, or rejects the review.

## Prompt Strategy

Prompts should ask for concise, coach-friendly output with strengths, improvement areas, recommended drills, and safety-aware language.

## Input/Output Format

Inputs include video ID, athlete context, session notes, coach notes, and optional transcript. Outputs include summary, observations, strengths, improvement areas, and drill recommendations.

## Coach Review Statuses

- `draft`
- `pending_coach_review`
- `approved`
- `rejected`
- `published`

## Endpoints

- `POST /reviews`
- `GET /reviews/{review_id}`
- `PATCH /reviews/{review_id}`
- `POST /reviews/{review_id}/approve`
- `POST /reviews/{review_id}/reject`
- `POST /reviews/{review_id}/publish`

## Tables Owned

- `ai_reviews`
- `ai_observations`
- `coach_reviews`
- `drill_recommendations`

## Testing Plan

- Review generation request validation
- Prompt construction
- AI provider response parsing
- Status transitions
- Coach edit and approval flow

## Future Improvements

- Background review jobs
- Model comparison
- Confidence scoring
- Custom sport-specific prompts
- Computer vision integration
# Timeline Outbox

The repository provides outbox persistence, a publisher, and safe event factories. Future review domain transactions must add requested/generated/failed and coach edited/approved/rejected rows before commit. Raw prompts and model output are forbidden from timeline metadata; approval is the transition to athlete-visible feedback.
