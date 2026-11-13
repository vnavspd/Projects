# Projects

Personal projects focused on systems that validate their outputs, surface failure modes early, and apply risk controls before problems compound.

## Prediction Market Research & Risk System

VPS-hosted, event-driven Python trading platform that combines real-time cryptocurrency and prediction-market data to identify pricing discrepancies. It evaluates multiple strategies concurrently and executes trades with integrated risk management and recovery mechanisms.

**Focus areas**
- Latency-conscious infrastructure. Hosted on a European VPS to reduce network latency to trading APIs, using asynchronous WebSocket feeds and concurrent processing for market monitoring and strategy evaluation.
- Multi-source signal validation. Integrated Binance spot prices, Polymarket order books, and Chainlink settlement-oracle data, with optional Coinbase/Kraken consensus checks to reject contradictory signals.
- Strategy development. Implemented complementary approaches spanning sum-to-one arbitrage, spot-price lag, spread discrepancies, and volatility/Z-score-based mispricing, alongside sports-result and sportsbook-odds strategies.
- Layered risk management. Applied trade-size and liquidity constraints, concurrent-position limits, correlated-asset exposure caps, daily and per-strategy loss limits, and circuit breakers monitoring drawdown, loss streaks, win-rate deterioration, and partial fills.
- Execution correctness. Developed fee-aware sizing, fresh-order-book verification, minimum-order checks, in-flight market reservations, fill-or-kill order handling, and emergency unwinding when only one leg of a trade fills.
- Operational observability. Built monitoring dashboards and structured logs that capture executions, outcomes, market context, and reasons for rejecting opportunities.

**Stack**  
Python, asyncio, aiohttp, WebSockets, REST APIs, Polymarket CLOB/Gamma/RTDS, Binance, Chainlink, Coinbase/Kraken feeds, Web3.py, Pydantic, FastAPI/Uvicorn, Loguru, Rich, pytest/pytest-asyncio, Linux VPS deployment with systemd.

**Reliability and evaluation notes**
- Configurable paper trading models slippage, failed fills, partial fills, execution delays, and latency jitter to exercise behavior before live execution.
- Historical-data replay with fee and slippage estimates plus drawdown reporting, and a calibration tool that estimates directional accuracy across momentum buckets using historical Binance candles.
- Bayesian outcome tracker grouped by entry conditions, with a minimum observation threshold, time-decayed observations, and an exploration allowance. Simulated trades are excluded from its learning data.
- Automatic WebSocket reconnection and resubscription, periodic atomic state saves, restart recovery, wallet and position reconciliation, and fallback resolution lookups across Polymarket APIs.
- Recorded expected versus realized outcomes and rejection reasons, allowing investigation of stale prices, vanished opportunities, execution failures, and risk-limit decisions.
- Automated tests cover execution failures, risk checks, circuit breakers, fee calculations, position tracking, market-data handling, and backtest reporting.

## Sublimest

Real-estate fair-price ML system built from more than 620,000 Canadian listing and historical Québec sale records. It uses ensemble models and over 200 property, geographic, assessment, temporal, neighbourhood, and market features to estimate a home’s expected sale price and identify potential overpricing or underpricing.

**Focus areas**
- Combined 334,000+ Québec property transactions with 287,000+ enriched Canadian listings.
- Matched sale and listing records through MLS identifiers, normalized addresses, geographic proximity, and property similarity.
- Prevented target leakage by removing sale-price-derived features and fitting geographic and market statistics exclusively on training data.
- Engineered more than 200 candidate features covering property characteristics, municipal assessments, seasonality, neighbourhood quality, geographic clusters, market activity, and data completeness.
- Recovered missing floor-area data, standardized inconsistent units and categories, validated geographic coordinates, and filtered extreme values.
- Used temporal train/test separation so the model is evaluated on later transactions rather than only random historical rows.
- Made large training runs resumable with staged checkpoints and integrity hashes.

**Stack**  
Python, pandas, NumPy, scikit-learn, XGBoost, LightGBM, Random Forest, Flask, joblib, pytest, Next.js, React, MongoDB, NextAuth, Stripe, Zod, Google Maps.

**Reliability and evaluation notes**
- Explicit leakage tests verify that changing `sale_amount` does not change engineered input features.
- Target-derived features such as sale-price-per-square-foot and assessment-to-sale-price ratios were identified and removed.
- City, neighbourhood, and geographic-cluster statistics are learned from the training partition and reused during evaluation.
- The sale-price pipeline supports a January 1, 2024 temporal cutoff, price-band evaluation, feature-importance analysis, and separate model metrics.
- XGBoost, LightGBM, and Random Forest predictions are combined through validation-derived ensemble weights.
- Long-running training jobs can resume from data-loading, feature-engineering, individual-model, or completed-ensemble checkpoints.

## ClairContrat

Production-grade, French-first AI document-intelligence SaaS for Québec. It ingests PDFs, DOCX files, and scanned images, performs OCR and multi-stage classification and extraction, and generates evidence-linked reports covering parties, clauses, obligations, risks, dates, amounts, and questions to investigate.

**Focus areas**
- LLM reliability and productization: deterministic source grounding, unsupported-claim filtering, prompt-injection guardrails, structured-output validation, privacy-safe evaluations, French-language consistency, secure document handling, transactional usage entitlements, idempotent billing, failure recovery, and model cost and latency observability.

**Stack**  
Next.js 15, React 19, TypeScript, OpenAI API, MongoDB transactions, Tesseract.js, Mammoth, PDF parsing, NextAuth, Stripe, Upstash Redis, Resend, AES-256-GCM, SHA-256, Vitest, Playwright, Axe, jsPDF, Vercel.

**Reliability and evaluation notes**
- Multi-stage AI pipeline for document validation, classification, structured extraction, category-specific analysis, deterministic post-processing, and report generation.
- Extracted amounts and dates must be supported by verbatim source excerpts; unsupported facts are removed before reports are saved.
- Safeguards against invented totals and double-counting current amounts, proposed amounts, fees, components, and explicitly stated totals.
- Uploaded document text is treated as untrusted data; models are instructed to ignore embedded commands or attempts to alter their role.
- Four synthetic, privacy-safe golden documents cover employment offers, invoices, rent renewals, and service contracts.
- Deterministic evaluation gate measures amount/date precision and recall, fabricated-amount rejection, and primary-amount accuracy. The current four-fixture gate scores 100% across those metrics.
- Opt-in end-to-end model evaluation with release thresholds of at least 95% precision, 85% recall, 100% fabricated-amount rejection, and 90% primary-amount accuracy.
- Analysis pipeline, extraction schema, prompts, grounding logic, model, and truncation metadata are versioned so reports are reproducible and model changes are auditable.
- Configurable provider timeouts and privacy-conscious telemetry for latency, token consumption, request status, and estimated CAD cost without recording document contents.
- Transactional quota and credit reservations so usage is committed only after successful report delivery, failed analyses refund credits exactly once, and concurrent requests cannot exceed limits.
- Scheduled recovery of interrupted analyses, idempotent Stripe webhook and Checkout processing, bounded retention jobs, health/readiness endpoints, structured redacted logging, and Redis-backed rate limiting.
- Document data protected with file-signature validation, authenticated ownership checks, AES-256-GCM encryption, SHA-256 integrity hashes, data export, per-analysis deletion, account deletion, and configurable retention.
- At review time, 101 active tests, deterministic evaluations, linting, type checking, secret scanning, and the 194-page production build passed.
