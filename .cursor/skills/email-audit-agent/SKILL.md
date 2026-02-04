---
name: email-audit-agent
description: Develop and extend the Personal Secure Email Audit Agent—a read-only Telegram bot for analyzing emails across Gmail, Yandex, Mail.ru. Use when working on email analysis, security features, learning engine, or Telegram bot integration.
---

# Email Audit Agent Development

## Quick Reference

**Project**: Personal Secure Email Audit Agent (read-only, no auto-actions)  
**Stack**: Python, Telegram Bot, IMAP/Gmail API, PostgreSQL/SQLite  
**Reference**: `Full_System_Prompt.txt` contains full architecture

## Iron Laws (Never Violate)

1. **Read-only**: No reply, forward, open links, download attachments, mark read/unread
2. **Network isolation**: No HTTP/DNS/API calls from email content
3. **Zero auto-execution**: No auto-archive, auto-trust, or "obvious" actions
4. **Security first**: When in doubt → safer option

## Architecture Layers

| Layer | Key Files |
|-------|-----------|
| `telegram_interface/` | bot.py, formatters.py, rate_limiter.py |
| `email_connectors/` | imap_client.py, gmail_api.py, yandex_imap.py, mailru_imap.py |
| `email_parser/` | sanitizer.py, header_analyzer.py, metadata_extractor.py |
| `risk_aggregator/` | scorer.py, conflict_resolver.py |
| `analysis_engine/` | rules.py, anti_phishing.py |
| `learning_engine/` | feature_extractor.py, predictor.py, established_decisions.py |
| `llm_explainer/` | safe_prompter.py, client.py |
| `storage/` | metadata_store.py, features_store.py, migrations/ |
| `security/` | encryption.py, secrets_manager.py, audit_log.py |

## Common Tasks

### Adding a new rule
- Add to `analysis_engine/rules.py` or `anti_phishing.py`
- Use deterministic checks only (no ML, no guessing)
- Return `RuleResult` with score, confidence, flags

### Adding a provider
- Create `*_imap.py` in `email_connectors/`
- Add to `EmailProvider` enum and `UnifiedEmailClient._init_client`
- Handle encoding quirks (e.g. Yandex KOI8-R, Mail.ru UTF-7)

### Modifying risk scoring
- Weights in `risk_aggregator/scorer.py` (default: rules 40%, stats 30%, learning 20%, LLM 10%)
- LLM used only when `_is_ambiguous()` (variance > 0.3 or confidence < 0.6)

### Adding learning features
- Extend `EmailFeatures` in `learning_engine/feature_extractor.py`
- Use hashed patterns only—no raw content
- Update `feature_weights` table schema if needed

## Code Style

- Type hints everywhere
- Docstrings on public classes/functions
- Use `logger`, never `print`
- Dependency injection for testability

## Security Checklist

- [ ] No raw email body in LLM prompts
- [ ] Sanitize subject for prompt injection
- [ ] Credentials encrypted at rest
- [ ] Audit log for decisions
- [ ] Telegram whitelist configured
