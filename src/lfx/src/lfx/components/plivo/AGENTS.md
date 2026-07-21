# Plivo components — agent context

Langflow components that call the Plivo REST API. Two components live here:

- `send_sms.py` — `PlivoSendSMSComponent` ("Plivo Send SMS"): `POST https://api.plivo.com/v1/Account/{AUTH_ID}/Message/`.
- `make_call.py` — `PlivoMakeCallComponent` ("Plivo Make Call"): `POST https://api.plivo.com/v1/Account/{AUTH_ID}/Call/`.

## Plivo contract (ground truth — do not change from memory)
- Auth is HTTP Basic: Auth ID as username, Auth Token as password. Console is `cx.plivo.com` (never `console.plivo.com`).
- REST base: `https://api.plivo.com/v1/Account/{AUTH_ID}/`.
- SMS body keys are `src`, `dst`, `text`, `type` ("sms"). Numbers are E.164. Multiple `dst` recipients are joined with `<`, not commas or arrays.
- SMS success is HTTP 202 Accepted (message queued, not delivered). Response carries `message_uuid` (a list) and `api_id`.
- Call body keys are `from`, `to`, `answer_url` (the SDK's `from_`/`to_`/`answer_url` map to these REST keys). Numbers are E.164.
- Call success is HTTP 201 Created (call fired/queued, not answered). Response carries `request_uuid` and `api_id`. Do not check for HTTP 200 on the call endpoint.

## Conventions
- Async `build_output` with `httpx.AsyncClient`, `raise_for_status`, and layered exception handling.
- Credentials default from the `PLIVO_AUTH_ID`, `PLIVO_AUTH_TOKEN`, and `PLIVO_SRC` environment variables and can be overridden per node in the UI. Never hardcode secrets.
- Registration lives in `src/lfx/src/lfx/components/__init__.py`; the `plivo` module is listed in the lazy-import block, the `_dynamic_imports` map, and `__all__`.

## Verify
Sending SMS or placing a call bills the account and is not idempotent. Test against a real Plivo account and capture screenshots of a queued message (202) and a fired call (201) before relying on it.
