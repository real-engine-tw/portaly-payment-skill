# Changelog

All notable changes to this skill will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this skill adheres to [Semantic Versioning](https://semver.org/).

## [0.7.0] - 2026-04-09

### Added
- External product checkout API documentation and workflow details
- Product bundles integration instructions for subscription plans
- Enhanced callback verification instructions for security and implementation clarity

## [0.6.0] - 2026-04-01

### Added
- Cursor integration installation instructions
- Subscription list and order query APIs with pagination and rate limiting details
- Subscriber self-service portal details for managing subscriptions
- Merchant profile config endpoint support for `merchantName` and image upload process

## [0.5.0] - 2026-03-25

### Added
- Dynamic pricing plans documentation and API request requirements
- Recurring subscription management and lifecycle APIs
- Cancellation policy clarification for recurring subscriptions

### Changed
- Clarified plan sharing behavior across live and test modes
- Added encoding notes for handling garbled text in plan names and descriptions

## [0.4.0] - 2026-03-23

### Added
- Sandbox environment details and test mode documentation
- API key modes (live/test) documentation

### Changed
- Refined descriptions and clarified sandbox environment details in SKILL.md
- Clarified `billingPeriod` options in subscription plans

## [0.3.0] - 2026-03-23

### Added
- Windows environment notes for PowerShell encoding issues

### Fixed
- Typo in API key application steps

### Changed
- Enhanced quick start guide and API key instructions for clarity

## [0.2.0] - 2026-03-20

### Added
- Config API guidance for Agent to auto-create plans (`feat: 加入 config API 引導 Agent 自動建立方案`)

### Changed
- Refined checkout and renewal documentation for clarity and completeness
- Enhanced documentation for Portaly Vibe Payment environments and API hosts
- Clarified references and guidelines for Portaly Vibe Payment integration

## [0.1.0] - 2026-03-20

### Added
- Initial skill definition (`SKILL.md`) with Portaly Vibe Payment integration instructions
- API contract reference (`references/api-contract.md`)
- Checkout and renewal reference (`references/checkout-and-renewal.md`)
- Callback signing scripts for Python and JavaScript (`scripts/`)
- README with installation instructions for Claude Code, Codex, and OpenClaw
